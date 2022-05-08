## opengauss存储模块之freelist.cpp解读（一）

<font color = red>**概述：freelist.cpp位于buffer目录下，作用是负责管理缓冲池替换策略**</font>

**控制信息量的定义，主要是retry时间**

```c++
const int MIN_DELAY_RETRY = 100;
const int MAX_DELAY_RETRY = 1000;
const int MAX_RETRY_TIMES = 1000;
const float NEED_DELAY_RETRY_GET_BUF = 0.8;
```



**BufferStrategyControl结构体定义**

BufferStrategyControl主要是记录共享freelist的控制信息

```c++
typedef struct BufferStrategyControl {
    /* 使用自旋锁来保护下面的值 */
    slock_t buffer_strategy_lock;

    /*
     * 获取一个实际的缓冲区，它需要被NBuffers取模使用
     */
    pg_atomic_uint32 nextVictimBuffer;

    /*
     * 统计数据，counter应该足够宽一保证不会溢出
     */
    uint32 completePasses;            /* 时钟扫描的完整周期 */
    pg_atomic_uint32 numBufferAllocs; /* 自上次重置以来已分配的缓冲区 */

    /*
     * Bgworker进程在活动时被通知，如果没有则为-1
     */
    int bgwprocno;
} BufferStrategyControl;
```

**StrategyDelayStatus结构体定义**

StrategyDelayStatus负责记录推迟状态

```c++
typedef struct {
    int64 retry_times;//retry时间
    int cur_delay_time;//推迟时间
} StrategyDelayStatus;
```

**函数PageListBackWrite**

PageListBackWrite的作用是把读取结果写回到页表

```c++
void PageListBackWrite(uint32* bufList, int32 n,
    /* 将要扫描的缓存列表 */
    uint32 flags = 0,                 /* 操作标志 */
    SMgrRelation use_smgrReln = NULL, /* 操作关联 */
    int32* bufs_written = NULL,       /* 操作写入计数返回 */
    int32* bufs_reusable = NULL);     /* 操作可重复使用计数返回 */
```

**函数perform_delay定义**

perform_delay的作用是延迟读取结果显示

```c++
static void perform_delay(StrategyDelayStatus *status)
{
    if (++(status->retry_times) > MAX_RETRY_TIMES &&
        get_dirty_page_num() > g_instance.attr.attr_storage.NBuffers * NEED_DELAY_RETRY_GET_BUF) {
        if (status->cur_delay_time == 0) {
            status->cur_delay_time = MIN_DELAY_RETRY;
        }
        pg_usleep(status->cur_delay_time);

        /* 将延迟增加1X到2X之间的一个随机分数 */
        status->cur_delay_time += (int)(status->cur_delay_time * ((double)random() / (double)MAX_RANDOM_VALUE) + 0.5);
        if (status->cur_delay_time > MAX_DELAY_RETRY) {
            status->cur_delay_time = MIN_DELAY_RETRY;
        }
    }
    return;
}
```

**函数ClockSweepTick定义**

<font color = red>作用：将时钟指针向前移动一个缓冲区，并返回当前缓冲区的id</font>>

```c++
static inline uint32 ClockSweepTick(int max_nbuffer_can_use)
{
    uint32 victim;

    /*
     * 如果有多个进程，则手动移动一个缓冲区,
	 * 这可能会导致缓冲区被返回稍微超出
	 * 明显的秩序。
     */
    victim = pg_atomic_fetch_add_u32(&t_thrd.storage_cxt.StrategyControl->nextVictimBuffer, 1);
    if (victim >= (uint32)max_nbuffer_can_use) {
        uint32 original_victim = victim;

        /* 总是包装我们在BufferDescriptors中查找的内容 */
        victim = victim % max_nbuffer_can_use;

        /*
         * 如果是我们造成了闭环，强迫
		 * completePasses在保持自旋锁时增加。我们
		 * 需要自旋锁，所以StrategySyncStart()可以返回一个一致的
		 * 由next受害缓冲区和completePasses组成。
         */
        if (victim == 0) {
            uint32 expected;
            uint32 wrapped;
            bool success = false;

            expected = original_victim + 1;

            while (!success) {
                /*
                 * 在增加completePasses的同时获取自旋锁。那
				 * 允许其他读取器读取nextVictimBuffer和
				 * 以一致的方式完成
				 * StrategySyncStart()。从理论上讲，延迟增量
				 * 可能导致next受害缓冲区溢出，但这是
				 * 不太可能的，也不会特别有害。
                 */
                SpinLockAcquire(&t_thrd.storage_cxt.StrategyControl->buffer_strategy_lock);

                wrapped = expected % max_nbuffer_can_use;

                success = pg_atomic_compare_exchange_u32(&t_thrd.storage_cxt.StrategyControl->nextVictimBuffer,
                                                         &expected, wrapped);
                if (success)
                    t_thrd.storage_cxt.StrategyControl->completePasses++;
                SpinLockRelease(&t_thrd.storage_cxt.StrategyControl->buffer_strategy_lock);
            }
        }
    }
    return victim;
}
```

