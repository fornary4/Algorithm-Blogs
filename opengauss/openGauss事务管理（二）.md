# openGauss事务管理（二）

### 数据库事务一致性的实现原理

#### openGauss采用 MVCC(多版本并发控制)机制来保证与写事务并发执行的查询事务的一致性。

MVCC（Multi-Version Concurrency Control，多版本并发控制）一种并发控制机制，在数据库中用来控制并发执行的事务，控制事务隔离进行。MVCC是通过保存数据在某个时间点的快照来进行控制的。使用MVCC就是允许同一个数据记录拥有多个不同的版本。然后在查询时通过添加相对应的约束条件，就可以获取用户想要的对应版本的数据。

**MVCC的基本机制是:写事务不会原地修改元组内容,而是将被修改的元组标记为这条记录的一个旧版本(标记xmax),同时插入一条修改后的元组,从而产生这条记录的一个新版本;对于在一个查询事务开始时还没有提交的写事务,那么这个查询事务始终认为该写事务没有提交。**

在 MVCC中,最关键的技术点有两个:

- **元组版本号的实现**
- **快照的实现**



**使用事务号实现元组版本号**

在openGauss中,采用全局递增的事务号来作为一个元组的版本号,每个写事务都会获得一个新的事务号。如上所述,一个元组的头部会记录两个事务号 xmin和xmax,分别对应元组的插入事务和删除(更新)事务。xmin和 xmax决定了元组的命期,亦即该版本的可见性窗口。

**使用活跃事务数组实现快照**

在数据库进程中,维护一个全局的数组,其中的成员为正在执行的事务信息,包括事务的事务号,该数组即活跃事务数组。在每个事务开始的时候,复制一份该数组内容。当事务执行过程中扫描到某个元组时,需要通过判断元组xmin和xmax这两个事务(即元组的插入事务和删除事务)对于查询事务的可见性,来决定该元组是否对查询事务可见。以xmin为例,首先查询CLOG,判断该事务是否提交,如果未提交,则不可见;如果提交,则进一步判断该xmin是否在查询事务的活跃事务数组中。如果xmin在该数组中,或者xmin的值大于该数组中事务号的最大值(事务号是全局递增发放的),那么该xmin事务一定在该查询事务开始之后才会提交,因此对于查询事务不可见;如果xmin不在该数组中,或者小于该数组中事务号的最小值,那么该xmin事务一定在该查询事务开始之前就已经提交,因此对于查询事务可见

![image-20211027001412047](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20211027001412047.png)

#### 快照snapshot源码分析

对于不同的快照类型,使用SnapshotData结构来表示,“常规”(MVCC)快照和“特殊”快照都有非MVCC语义。快照的特定语义由其类型编码,每种类型的快照的行为应与它一起记录enum值，最好不是特定于单个表AM。

**快照模式定义**

如果元组对给定的MVCC快照有效，则该元组是可见的。

```c++
typedef enum SnapshotSatisfiesMethod
{
    
    SNAPSHOT_MVCC = 0,

    
    SNAPSHOT_LOCAL_MVCC,

   
    SNAPSHOT_NOW,

   
    SNAPSHOT_SELF,

    
    SNAPSHOT_ANY, //任何元组

   
    SNAPSHOT_TOAST,

   
    SNAPSHOT_DIRTY,

   
    SNAPSHOT_HISTORIC_MVCC,
} SnapshotSatisfiesMethod;
```

**可见性结果定义**

```c++
typedef enum TM_Result
{
    /*
     *表示操作成功
     */
    TM_Ok,

    /* 受影响的元组对相关快照不可见 */
    TM_Invisible,

    /* 调用后端已经修改了受影响的元组 */
    TM_SelfModified,

    /*
     * 受影响的元组由另一个事务更新
     */
    TM_Updated,

    /* 受影响的元组已被另一个事务删除 */
    TM_Deleted,

    /*
     * 受影响的元组目前正在被另一个会话修改。这
	 * 只在(update/delete/lock)_tuple被指示为not时返回
     */
    TM_BeingModified,
	TM_SelfCreated,
	TM_SelfUpdated
} TM_Result;
```

