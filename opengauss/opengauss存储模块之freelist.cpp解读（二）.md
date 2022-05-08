## opengauss存储模块之freelist.cpp解读（二）

### **缓冲区替换策略分析**

<font color = red>当缓冲池里没有可用的页面时，缓冲区管理器要使用某种策略把某些页面的数据写回磁盘，腾出自由页面以便保存后面的读写操作的数据，这一过程称为页面置换。</font>

openGuass使用一个专门的进程，采用时钟算法进行页面置换。它为每个缓冲区设置一个计数器，每隔一段时间则顺序扫描缓冲池里的每一个缓冲区，检查计数器。如果计数器为零，则说明这一缓冲区可回收使用。于是，系统先将缓冲区内脏数据写入磁盘，而后将缓中区放入自由缓冲区列表。如果计数器的值不为零，系统则将计数器的值减少。而当某个进程访问缓冲区的数据时，该缓冲区的计数器则会增加。

### **重要函数分析**

**StrategyGetBuffer（实现缓冲区动态替换）**

作用：由bufmgr调用，以获取要使用的下一个候选缓冲区，BufferAlloc()的唯一硬性要求是

所选缓冲区当前不能被任何进程固定。

strategy是一个BufferAccessStrategy对象，默认策略为NULL。

为了确保没有人能在我们之前锁定缓冲区，我们必须这样做，返回buffer报头自旋锁仍然保持的缓冲区。

```c++
BufferDesc* StrategyGetBuffer(BufferAccessStrategy strategy, uint32* buf_state)
{
    BufferDesc *buf = NULL;
    int bgwproc_no;
    int try_counter;
    uint32 local_buf_state = 0; /* 避免重复(取消)引用 */
    int max_buffer_can_use;
    bool am_standby = RecoveryInProgress();
    StrategyDelayStatus retry_lock_status = { 0, 0 };
    StrategyDelayStatus retry_buf_status = { 0, 0 };

    /*
     * 如果给定一个策略对象，看看它是否可以选择一个缓冲区
     */
    if (strategy != NULL) {
        buf = GetBufferFromRing(strategy, buf_state);
        if (buf != NULL) {
            return buf;
        }
    }

    /*
     * 如果被问及，我们需要唤醒bgwriter。因为我们不想依靠
	 * 自旋锁，我们强制从共享内存读取一次，然后
	 * 根据这个值设置锁存器。我们需要穿过这个长度
	 * 因为否则bgprocno可能会在我们检查的时候被重置
 	 * 编译器可能只是从内存中重新读取。
     */
    bgwproc_no = INT_ACCESS_ONCE(t_thrd.storage_cxt.StrategyControl->bgwprocno);
    if (bgwproc_no != -1) {
        /* 在设置锁存器之前，首先重置bgwprocno */
        t_thrd.storage_cxt.StrategyControl->bgwprocno = -1;

        /*
         * 这里没有获取ProcArrayLock，这有点麻烦。这是
		 * 实际上很好，因为procLatch从来没有被释放过，所以我们可以
		 * 可能设置错误的进程'(或没有进程')锁存。
         */
        SetLatch(&g_instance.proc_base_all_procs[bgwproc_no]->procLatch);
    }

    /*
     * 我们计算缓冲区分配请求，以便bgwriter可以估算
	 * 缓冲区消耗的速率。请注意，由
	 * 策略对象故意不在这里计算。
     */
    (void)pg_atomic_fetch_add_u32(&t_thrd.storage_cxt.StrategyControl->numBufferAllocs, 1);

    /* 检查候选列表 */
    if (g_instance.attr.attr_storage.enableIncrementalCheckpoint &&
        g_instance.bgwriter_cxt.bgwriter_num > 0) {
        buf = get_buf_from_candidate_list(strategy, buf_state);
        if (buf != NULL) {
            (void)pg_atomic_fetch_add_u64(&g_instance.bgwriter_cxt.get_buf_num_candidate_list, 1);
            return buf;
        }
    }

retry:
    /* 没有自由列表，所以运行“时钟扫描”算法 */
    if (am_standby)
        max_buffer_can_use =
            int(g_instance.attr.attr_storage.NBuffers * u_sess->attr.attr_storage.shared_buffers_fraction);
    else
        max_buffer_can_use = g_instance.attr.attr_storage.NBuffers;
    try_counter = max_buffer_can_use;
    int try_get_loc_times = max_buffer_can_use;
    for (;;) {
        buf = GetBufferDescriptor(ClockSweepTick(max_buffer_can_use));
        /*
         * 如果缓冲区是固定的，则不能使用它
         */
        if (!retryLockBufHdr(buf, &local_buf_state)) {
            if (--try_get_loc_times == 0) {
                ereport(WARNING,
                        (errmsg("try get buf headr lock times equal to maxNBufferCanUse when StrategyGetBuffer")));
                try_get_loc_times = max_buffer_can_use;
            }
            perform_delay(&retry_lock_status);
            continue;
        }

        retry_lock_status.retry_times = 0;
        if (BUF_STATE_GET_REFCOUNT(local_buf_state) == 0 &&
            (backend_can_flush_dirty_page() || !(local_buf_state & BM_DIRTY))) {
            /* 找到一个可用的缓冲区 */
            if (strategy != NULL)
                AddBufferToRing(strategy, buf);
            *buf_state = local_buf_state;
            (void)pg_atomic_fetch_add_u64(&g_instance.bgwriter_cxt.get_buf_num_clock_sweep, 1);
            return buf;
        } else if (--try_counter == 0) {
            /*
             * 我们已经扫描了所有的缓冲区，没有做任何状态改变
             */
            UnlockBufHdr(buf, local_buf_state);

            if (am_standby && u_sess->attr.attr_storage.shared_buffers_fraction < 1.0) {
                ereport(WARNING, (errmsg("no unpinned buffers available")));
                u_sess->attr.attr_storage.shared_buffers_fraction =
                    Min(u_sess->attr.attr_storage.shared_buffers_fraction + 0.1, 1.0);
                goto retry;
            } else if (dw_page_writer_running()) {
                ereport(LOG, (errmsg("double writer is on, no buffer available, this buffer dirty is %u, "
                                     "this buffer refcount is %u, now dirty page num is %ld",
                                     (local_buf_state & BM_DIRTY), BUF_STATE_GET_REFCOUNT(local_buf_state),
                                     get_dirty_page_num())));
                perform_delay(&retry_buf_status);
                goto retry;
            } else if (t_thrd.storage_cxt.is_btree_split) {
                ereport(WARNING, (errmsg("no unpinned buffers available when btree insert parent")));
                goto retry;
            } else
                ereport(ERROR, (errcode(ERRCODE_INVALID_BUFFER), (errmsg("no unpinned buffers available"))));
        }
        UnlockBufHdr(buf, local_buf_state);
        perform_delay(&retry_buf_status);
    }

    /* 不会出现的情况 */
    return NULL;
}
```

**StrategySyncStart（缓冲区同步）**

告诉BufferSync从哪里开始同步

结果是首先要同步的最佳缓冲区的缓冲区索引，BufferSync()将从那里循环读取缓冲区数组。

此外，我们返回完成传递计数(这实际上是next缓冲区的高阶位)和最近缓冲区的计数

如果传递的是非null指针，则分配，之后重新设置alloc计数

```c++
int StrategySyncStart(uint32 *complete_passes, uint32 *num_buf_alloc)
{
    uint32 next_victim_buffer;
    int result;

    SpinLockAcquire(&t_thrd.storage_cxt.StrategyControl->buffer_strategy_lock);
    next_victim_buffer = pg_atomic_read_u32(&t_thrd.storage_cxt.StrategyControl->nextVictimBuffer);
    result = next_victim_buffer % g_instance.attr.attr_storage.NBuffers;

    if (complete_passes != NULL) {
        *complete_passes = t_thrd.storage_cxt.StrategyControl->completePasses;
        /*
         * 另外添加之前发生的环绕次数
		 * completePasses可以增加,由ClockSweepTick()引用。
         */
        *complete_passes += next_victim_buffer / (unsigned int)g_instance.attr.attr_storage.NBuffers;
    }

    if (num_buf_alloc != NULL) {
        *num_buf_alloc = pg_atomic_exchange_u32(&t_thrd.storage_cxt.StrategyControl->numBufferAllocs, 0);
    }
    SpinLockRelease(&t_thrd.storage_cxt.StrategyControl->buffer_strategy_lock);
    return result;
}
```

**同步机制分析**

<font color = red>对于缓冲区内那些通过较高代价产生的对象，系统使用具有较高参考价值的计数器。在顺序扫描时，这些计数器的值不是简单的减少，而是以某种策略缩小（如除以4），这样可以保证它不会很快变成零，从而可以在内存中贮存较长时间。</font>

