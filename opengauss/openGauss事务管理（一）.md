# openGauss事务管理（一）

**数据库是一个共享资源，可以提供多个用户使用。**这些用户程序可以一个一个地串行执行，每个时刻只有一个用户程序运行，执行对数据库的存取，其他用户程序必须等到这个用户程序结束以后方能对数据库存取。但是如果一个用户程序涉及大量数据的输入/输出交换，则数据库系统的大部分时间处于闲置状态。因此，为了充分利用数据库资源，发挥数据库共享资源的特点，应该允许多个用户并行地存取数据库。但这样就会产生多个用户程序并发存取同一数据的情况，若对并发操作不加控制就可能会存取和存储不正确的数据，破坏数据库的一致性，所以数据库管理系统必须提供并发控制机制。**并发控制机制的好坏是衡量一个数据库管理系统性能的重要标志之一。**

#### 事务介绍

**并发控制是以事务（transaction）为单位进行的。事务是数据库的逻辑工作单位，它是用户定义的一组操作序列。**一个事务可以是一组SQL语句、一条SQL语句或整个程序。事务的开始和结束都可以由用户显示的控制，如果用户没有显式地定义事务，则由数据库系统按缺省规定自动划分事务。事务应该具有4种属性：原子性、一致性、隔离性和持久性。

1. **原子性：**事务的原子性保证事务包含的一组更新操作是原子不可分的，也就是说这些操作是一个整体，对数据库而言全做或者全不做，不能部分的完成。
2. **一致性：**一致性要求事务执行完成后，将数据库从一个一致状态转变到另一个一致状态。它是一种以一致性规则为基础的逻辑属性，例如在转账的操作中，各账户金额必须平衡。
3. **隔离性：**隔离性意味着一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。它要求即使有多个事务并发执行，看上去每个成功事务按串行调度执行一样。
4. **持久性：**系统提供的持久性保证要求一旦事务提交，那么对数据库所做的修改将是持久的，无论发生何种机器和系统故障都不应该对其有任何影响。

#### openGauss事务模型

openGauss是一个分布式的数据库。同样的,openGauss的事务机制也是一个从单机到分布式的双层构架。下图为openGauss集群事务组件构成示意图。

![image-20211026223543419](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20211026223543419.png)

**在openGauss集群中,事务的执行和管理主要涉及 GTM、CN 和DN 三种组件,其中:**

(1)GTM(GlobalTransaction Manager,全局事务管理器),负责全局事务号的分发、事务提交时间戳的分发以及全局事务运行状态的登记。对于采用多版本并发控制(Multi-Version Concurrency Control,MVCC)的事务模型,GTM 本质上可以简化为一个递增序列号(或时间戳)生成器,其为集群的所有事务进行了全局的统一排序,以确定快照(Snapshot)内容并由此决定事务可见性。在本文openGauss并发控制中,将进一步详述 GTM的作用。

(2)CN(CoordinatorNode,协调者节点),负责管理和推进一个具体事务的执行流程,维护和推进事务执行的事务块状态机。

(3)DN(DataNode,数据节点),负责一个具体事务在某一个数据分片内的所有读写操作。本文主要介绍显式事务和隐式事务执行流程中,CN 和 DN 上事务块状态机的推演,以及单机事务和分布式事务的异同。

#### 事务源码解析

**以WLMTransacton为例，在instr_workload.h头文件中，详细定义了WLM事务**



**状态数据,定义了最大值、最小值、平均值和总量，其中 TimestampTz是Double的类型定义**

```c++
typedef struct StatData {
    TimestampTz max;
    TimestampTz min;
    TimestampTz average;
    TimestampTz total;
} StatData;
```

**事务信息**

```c++
typedef struct WLMTransactonInfo {
    uint64 commit_counter;//提交计算器
    uint64 rollback_counter;//回滚计数器
    StatData responstime;//响应时间
} WLMTransactionInfo;
```

**初始化WLM事务的函数,作用是创建负载事务的hashtable，并添加事务**

```c++
void InitInstrWorkloadTransaction(void)
{
    if (!(IS_PGXC_COORDINATOR || IS_SINGLE_NODE)) {
        return;
    }
    LWLockAcquire(InstrWorkloadLock, LW_EXCLUSIVE);
    if (g_instance.stat_cxt.workload_info_hashtbl != NULL) {
        LWLockRelease(InstrWorkloadLock);
        return;
    }

    /* 创建hash表 */
    init_instr_workload_hashtbl();

    /* 为工作负载事务初始化所有用户条目 */
    InitInstrWorkloadTransactionUser();

    LWLockRelease(InstrWorkloadLock);
}
```

