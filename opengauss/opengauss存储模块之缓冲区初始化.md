## opengauss存储模块之缓冲区初始化

**概述：buf_init.cpp位于存储模块的buffer目录下，其作用是负责初始化缓冲区**

 **实现机制分析：**

1. 使用pageQueue作为缓冲区的容器，并确定了个page的优先级
2. 缓冲区位于freelist和查找数据结构中，缓冲区不能被替换, 由数据管理器或在IO期间使用
3. 缓冲必须是可用于在IO开始前查找,否则第二个进程试图读取缓冲区分配给它自己的副本和缓冲池将变得不一致。



**常量定义**

初始化pagequeue的长度为5

```c++
const int PAGE_QUEUE_SLOT_MULTI_NBUFFERS = 5;
```

**函数分析**

**MemsetPageQueue：重置缓冲区**

```c++
static void MemsetPageQueue(char *buffer, Size len)
{
    int rc;
    while (len > 0) {
        if (len < SECUREC_MEM_MAX_LEN) { //如果小于最大可用长度就扩展缓冲区
            rc = memset_s(buffer, len, 0, len);
            securec_check(rc, "", "");
            return;
        } else {  //去除溢出的部分
            rc = memset_s(buffer, SECUREC_MEM_MAX_LEN, 0, SECUREC_MEM_MAX_LEN);
            securec_check(rc, "", "");
            len -= SECUREC_MEM_MAX_LEN;
            buffer += SECUREC_MEM_MAX_LEN;
        }
    }
}
```

**InitBufferPool：初始化缓冲池**

```c++
void InitBufferPool(void)
{
    bool found_bufs = false;
    bool found_descs = false;
    bool found_buf_ckpt = false;
    uint64 buffer_size;

    t_thrd.storage_cxt.BufferDescriptors = (BufferDescPadded *)CACHELINEALIGN(
        ShmemInitStruct("Buffer Descriptors",
                        g_instance.attr.attr_storage.NBuffers * sizeof(BufferDescPadded) + PG_CACHE_LINE_SIZE,
                        &found_descs));

    /* 初始化候选缓冲区列表和候选缓冲区空闲映射 */
    candidate_buf_init();

#ifdef __aarch64__
    buffer_size = g_instance.attr.attr_storage.NBuffers * (Size)BLCKSZ + PG_CACHE_LINE_SIZE;
    t_thrd.storage_cxt.BufferBlocks =
        (char *)CACHELINEALIGN(ShmemInitStruct("Buffer Blocks", buffer_size, &found_bufs));
#else
    buffer_size = g_instance.attr.attr_storage.NBuffers * (Size)BLCKSZ;
    t_thrd.storage_cxt.BufferBlocks = (char *)ShmemInitStruct("Buffer Blocks", buffer_size, &found_bufs);
#endif

    if (BBOX_BLACKLIST_SHARE_BUFFER) {
        bbox_blacklist_add(SHARED_BUFFER, t_thrd.storage_cxt.BufferBlocks, buffer_size);
    }

    /*
     * 用于对被检查点的缓冲区id进行排序的数组,位于其中的共享内存，以避免分配大量的
	 * 内存。因为在检查点中间，或者检查指针重新启动时，内存分配将失败
     */
    g_instance.ckpt_cxt_ctl->CkptBufferIds =
        (CkptSortItem *)ShmemInitStruct("Checkpoint BufferIds",
                                        g_instance.attr.attr_storage.NBuffers * sizeof(CkptSortItem), &found_buf_ckpt);

    if (g_instance.attr.attr_storage.enableIncrementalCheckpoint && g_instance.ckpt_cxt_ctl->dirty_page_queue == NULL) {
        g_instance.ckpt_cxt_ctl->dirty_page_queue_size = g_instance.attr.attr_storage.NBuffers *
                                                         PAGE_QUEUE_SLOT_MULTI_NBUFFERS;
        MemoryContext oldcontext = MemoryContextSwitchTo(g_instance.increCheckPoint_context);

        Size queue_mem_size = g_instance.ckpt_cxt_ctl->dirty_page_queue_size * sizeof(DirtyPageQueueSlot);
        g_instance.ckpt_cxt_ctl->dirty_page_queue =
            (DirtyPageQueueSlot *)palloc_huge(CurrentMemoryContext, queue_mem_size);
        if (g_instance.ckpt_cxt_ctl->dirty_page_queue == NULL) {
            ereport(ERROR, (errmodule(MOD_INCRE_CKPT), errmsg("Memory allocation failed.\n")));
        }

        MemsetPageQueue((char*)g_instance.ckpt_cxt_ctl->dirty_page_queue, queue_mem_size);
        (void)MemoryContextSwitchTo(oldcontext);
    }

    if (found_descs || found_bufs || found_buf_ckpt) {
        /* 两者都应该出现，或者都不出现 */
        Assert(found_descs && found_bufs && found_buf_ckpt);
        /* 此路径仅在EXEC_BACKEND情况下使用 */
    } else {
        int i;

        /*
         * 初始化所有缓冲区头部信息。
         */
        for (i = 0; i < g_instance.attr.attr_storage.NBuffers; i++) {
            BufferDesc *buf = GetBufferDescriptor(i);
            CLEAR_BUFFERTAG(buf->tag);

            pg_atomic_init_u32(&buf->state, 0);
            buf->wait_backend_pid = 0;

            buf->buf_id = i;
            buf->io_in_progress_lock = LWLockAssign(LWTRANCHE_BUFFER_IO_IN_PROGRESS);
            buf->content_lock = LWLockAssign(LWTRANCHE_BUFFER_CONTENT);
            pg_atomic_init_u64(&buf->rec_lsn, InvalidXLogRecPtr);
            buf->dirty_queue_loc = PG_UINT64_MAX;
        }
    }

    /* 始化其他共享缓冲区管理内容 */
    StrategyInitialize(!found_descs);

    /* Init Vector Buffer管理 */
    DataCacheMgr::NewSingletonInstance();

    /* Init元数据缓存管理 */
    MetaCacheMgr::NewSingletonInstance();

    /* 初始化后端文件刷新内容 */
    WritebackContextInit(t_thrd.storage_cxt.BackendWritebackContext, &u_sess->attr.attr_common.backend_flush_after);
}
```

**add_size：对两个Size值相加，检查是否溢出**

```c++
Size add_size(Size s1, Size s2)
{
    Size result;

    result = s1 + s2;

    /* 假设Size是unsigned类型*/
    if (result < s1 || result < s2) //如果移除就报告错误
        ereport(
            ERROR, (errcode(ERRCODE_PROGRAM_LIMIT_EXCEEDED), errmsg("requested shared memory size overflows size_t")));

    return result;
}
```

**BufferShmemSize：获取缓冲区共享内存的大小**

计算包括数据页、缓冲区描述符、哈希表在内的缓冲池的共享内存大小。

```c++
Size BufferShmemSize(void)
{
    Size size = 0;

    /* 缓冲区描述符的大小 */
    size = add_size(size, mul_size(g_instance.attr.attr_storage.NBuffers, sizeof(BufferDescPadded)));
    size = add_size(size, PG_CACHE_LINE_SIZE);

    /* 数据页大小 */
    size = add_size(size, mul_size(g_instance.attr.attr_storage.NBuffers, BLCKSZ));
#ifdef __aarch64__
    size = add_size(size, PG_CACHE_LINE_SIZE);
#endif
    /* 由freelist.c控制的部分的大小 */
    size = add_size(size, StrategyShmemSize());

    /* bufmgr.c中检查点排序数组的大小 */
    size = add_size(size, mul_size(g_instance.attr.attr_storage.NBuffers, sizeof(CkptSortItem)));

    /* 候选缓冲区的大小 */
    size = add_size(size, mul_size(g_instance.attr.attr_storage.NBuffers, sizeof(Buffer)));

    /* 候选哈希表的大小 */
    size = add_size(size, mul_size(g_instance.attr.attr_storage.NBuffers, sizeof(bool)));

    return size;
}
```



**总结：**

**buf_init.cpp利用pagequeue来管理缓冲区，设置缓冲池来提高缓存效率，通过内存共享来实现数据交互，并实现各种操作**

