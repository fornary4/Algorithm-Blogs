## opengauss死锁处理



### **死锁定义**

在多道程序系统中，由于多个程序并发执行，改善了系统资源的利用率并提高了系统的处理能力。然后多个进程的并发执行也带来了新的问题-----死锁。所谓死锁是指多个进程因竞争资源而造成的一种僵局（相互等待），若无外力作用，这些进程都将无法向前推进。

**死锁产生的必要条件**

-  互斥：进程请求的 资源同时只允许一个进程占用。

-  请求和保持：进程已经持有了至少一个资源，但同时又提出了新的资源请求，此时对自己持有的资源保持不放。

-  不可被抢占：进程已经获得的资源不可被抢占，只能在进程使用完时由自己释放。

-  循环等待：产生死锁的一组进程必然存在一个资源占有的循环链。也就是：一组进程{A, B}，A在等B占有的资源释放，B在等A占有的资源释放。

  

首先引入一个概念，==系统资源分配图==（SRAG）

**资源分配图是一张有向图，一个系统资源分配图SRAG (System Resource Allocation Graph)可定义为一个二元组**，即SRAG= (V, E)，其中V是顶点的集合，而E是有向边的集合。顶点集合可分为两种部分：P= (P1,P2,…,Pn)，是由系统内的所有进程组成的集合，每一个P代表一个进程；R = (r1,r2,…,rn)，是系统内所有资源组成的集合， 每一个r代表一类资源。

在有向图中，用圆圈表示进程，用方框表示每类资源。每一类资源r,可能有多个实例， 可用方框中的圆点（实心圆点）表示各个资源实例。申请边为从进程到资源的有向边，表示进程申请一个资源，但当前该进程在等待该资源。分配边为从资源到进程的有向边，表示 有一个资源实例分配给进程。注意：一条申请边仅指向代表资源类的方框，表示申请时不 指定哪一个资源实例，而分配边必须由方框中的圆点引出，表明哪一个资源实例已被占有。

![image-20210818150657718](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210818150657718.png)

**在图c中，出现了环路，又因为每类资源只有一个，所有出现了死锁**

**在图d中，因为每类资源不止一个，所以没有出现死锁**



### **opengauss死锁处理思路**

<font color=red>首先检测系统的进程和资源使用情况，生成系统资源分配图，递归地检测是否存在环路，若出现环路，则生成一个锁的等待队列，对队列进行拓扑排序，以消除死锁。</font>



### 源码分析



**资源分配图边的定义**

```c++
typedef struct EDGE {
    PGPROC *waiter;  /* 正在等待的进程 */
    PGPROC *blocker; /* 它所等待的进程 */
    int pred;        
    int link;        
} EDGE;
```

**死锁信息**

```c++
typedef struct DEADLOCK_INFO {
    LOCKTAG locktag;   /* 等待的锁对象ID */
    LOCKMODE lockmode; /* 等候锁的类型 */
    ThreadId pid;      /* 阻塞进程的PID */
} DEADLOCK_INFO;
```

**等候队列的排列顺序**

```c++
typedef struct WAIT_ORDER {
    LOCK *lock;     /* 描述其等待队列的锁 */
    PGPROC **procs; /* PGPROC 数组新的顺序 */
    int nProcs;
} WAIT_ORDER;
```

**函数功能分析**

```c++
/* 检测检测是否处于死锁状态 */
DeadLockState DeadLockCheck(PGPROC *proc);

/* 递归地检测系统死锁，为在此级别检测到的任何循环添加每个可能的解决方案约束。
 *如果不存在解决方案，则返回 TRUE，
 *如果可以获得无死锁状态，则返回 FALSE */
bool DeadLockCheckRecurse(PGPROC *proc);

/* 递归地检测资源分配图的环路 */
bool FindLockCycleRecurse(PGPROC *checkProc, int depth, EDGE *softEdges, int *nSoftEdges);

/* 测试排序后的队列是否能解决死锁 */
int TestConfiguration(PGPROC *startProc);

/* 在后端启动时初始化死锁检查器 */
void InitDeadLockChecking(void);
```



**拓扑排序实现**

等待队列的拓扑排序 生成一个锁的等待队列的重新排序，该队列满足关于某些进程在其他进程之前的给定约束。 每个这样的约束都是部分有序的一个实例，最小化不需要实现部分有序的队列的重新排列。

```c++
static bool TopoSort(LOCK *lock, EDGE *constraints, int nConstraints, PGPROC **ordering) 
{
    PROC_QUEUE *waitQueue = &(lock->waitProcs);
    int queue_size = waitQueue->size;
    PGPROC *proc = NULL;
    int i, j, k, last;

    /* 首先，以当前顺序填充topoProcs数组 */
    proc = (PGPROC *)waitQueue->links.next;
    for (i = 0; i < queue_size; i++) {
        t_thrd.storage_cxt.topoProcs[i] = proc;
        proc = (PGPROC *)proc->links.next;
    }

    /*
     *扫描约束，并为数组中的每个 proc，生成表示它必须在其他事物之前的约束数量的计数，以及表明它必须在其他事物之后的约束列表。
     *第 j 个 proc 的计数存储在 beforeConstraints[j] 中，其列表的头部存储在 afterConstraints[j] 中。
     *每个约束将其列表链接存储在约束 constraints[i].link（注意任何约束都将仅在一个列表中）。
     *在constraints[i].pred 中记住了第i 个约束的before-proc 的数组索引。
     */
    errno_t ret = memset_s(t_thrd.storage_cxt.beforeConstraints, queue_size * sizeof(int), 0, queue_size * sizeof(int));
    securec_check(ret, "\0", "\0");
    ret = memset_s(t_thrd.storage_cxt.afterConstraints, queue_size * sizeof(int), 0, queue_size * sizeof(int));
    securec_check(ret, "\0", "\0");
    for (i = 0; i < nConstraints; i++) {
        proc = constraints[i].waiter;
        /* 如果不是这个锁，则忽略约束 */
        if (proc->waitLock != lock) {
            continue;
        }
        /* 在数组中找到waiter进程 */
        for (j = queue_size; --j >= 0;) {
            if (t_thrd.storage_cxt.topoProcs[j] == proc) {
                break;
            }
        }
        Assert(j >= 0); /* 应该找到匹配到的 */
        /* 在数组中找到阻塞程序 */
        proc = constraints[i].blocker;
        for (k = queue_size; --k >= 0;) {
            if (t_thrd.storage_cxt.topoProcs[k] == proc) {
                break;
            }
        }
        Assert(k >= 0);                            /* 应该找到匹配到的 */
        t_thrd.storage_cxt.beforeConstraints[j]++;
        /* 将此约束添加到拦截器的后约束列表中 */
        constraints[i].pred = j;
        constraints[i].link = t_thrd.storage_cxt.afterConstraints[k];
        t_thrd.storage_cxt.afterConstraints[k] = i + 1;
    }
    /* 
     * 现在向后扫描 topoProcs 数组，
     * 需要避免冗余搜索
     */
    last = queue_size - 1;
    for (i = queue_size; --i >= 0;) {
        /* 查找下一个要输出的候选项 */
        while (t_thrd.storage_cxt.topoProcs[last] == NULL) {
            last--;
        }
        for (j = last; j >= 0; j--) {
            if (t_thrd.storage_cxt.topoProcs[j] != NULL && t_thrd.storage_cxt.beforeConstraints[j] == 0) {
                break;
            }
        }
        /* 如果没有可用的候选项，则拓扑排序失败 */
        if (j < 0) {
            return false;
        }
        /* 输出候选项，并通过对topoProcs[]条目进行归零来标记它已完成 */
        ordering[i] = t_thrd.storage_cxt.topoProcs[j];
        t_thrd.storage_cxt.topoProcs[j] = NULL;
        for (k = t_thrd.storage_cxt.afterConstraints[j]; k > 0; k = constraints[k - 1].link) {
            t_thrd.storage_cxt.beforeConstraints[constraints[k - 1].pred]--;
        }
    }

    /* 完成 */
    return true;
}
```

