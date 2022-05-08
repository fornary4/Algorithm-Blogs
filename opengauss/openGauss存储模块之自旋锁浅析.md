## openGauss存储模块之自旋锁浅析

### 自旋锁的定义

<font color = red>自旋锁（spinlock）：是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。</font>

获取锁的线程一直处于活跃状态，但是并没有执行任何有效的任务，使用这种锁会造成busy-waiting。

自旋锁是为实现保护共享资源而提出一种锁机制。其实，自旋锁与互斥锁比较类似，它们都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个保持者，也就说，在任何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同。对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，”自旋”一词就是因此而得名。

### 概述

spin.cpp位于存储模块的lmgr模块下，作用是从硬件层面独立实现自旋锁。

对于具有test-and-set (TAS)指令的机器，需要使用s_lock.h/.c定义自旋锁实现。这个文件只包含一个存根，使用pgsemaphres实现自旋锁。除非信号量以一种不涉及内核调用的方式实现，但这种方式的速度太慢，无法发挥很大作用。

### 代码解析

```c++
#include "postgres.h"
#include "knl/knl_variable.h"

#include "miscadmin.h"
#include "replication/walsender.h"
#include "storage/lock/lwlock.h"
#include "storage/spin.h"

#ifdef HAVE_SPINLOCKS

/*
 * 报告支持自旋锁所需的信号量。
 */
int SpinlockSemas(void)
{
    return 0;
}
#else /* 已经拥有自旋锁 */

/*
 * 没有TAS，所以自旋锁被实现为pgsemaphres
 *
 * 报告支持自旋锁所需的信号量
 */
int SpinlockSemas(void)
{
    int nsemas;

    /*
     * 将此逻辑分发到受影响的模块中会更干净，
	 * 类似于shmem空间估计的处理方式。
     *
     * 不过现在，自旋锁的用户还不够多
	 * 把信息留在这里。
     */
    nsemas = NumLWLocks();                                      /* 每个lwlock对应一个 */
    nsemas += g_instance.attr.attr_storage.NBuffers;            /* 每个缓冲区报头一个 */
    nsemas += 2 * g_instance.attr.attr_storage.max_wal_senders; /* 每个wal和数据发送进程有一个 */
    nsemas += 30;                                               /* 再加上一堆用于其他小规模用途 */

    return nsemas;
}

/*
 * s_lock.h hardware-spinlock仿真
 */
void s_init_lock_sema(volatile slock_t *lock)
{
    PGSemaphoreCreate((PGSemaphore)lock);
}

void s_unlock_sema(volatile slock_t *lock)
{
    PGSemaphoreUnlock((PGSemaphore)lock);
}

bool s_lock_free_sema(volatile slock_t *lock)
{
    /* 我们目前不使用S_LOCK_FREE */
    ereport(ERROR, (errcode(ERRCODE_FEATURE_NOT_SUPPORTED), errmsg("spin.c does not support S_LOCK_FREE()")));
    return false;
}

int tas_sema(volatile slock_t *lock)
{
    /* 注意，如果*success*， TAS宏返回0 */
    return !PGSemaphoreTryLock((PGSemaphore)lock);
}

#endif /* 已经拥有自旋锁 */

```

