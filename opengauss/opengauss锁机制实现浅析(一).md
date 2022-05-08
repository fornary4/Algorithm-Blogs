## opengauss锁机制实现浅析（一）



**什么是锁**

当并发事务同时访问一个资源时，有可能导致数据不一致，因此需要一种机制来将数据访问顺序化，==以保证数据库数据的一致性==。锁就是其中的一种机制。数据库锁设计的初衷是处理并发问题。作为多用户共享的资源，当出现并发访问的时候，为了保证数据的一致性，数据库需要合理地控制资源的访问规则。在计算机科学中，锁是在执行多线程时用于强行限制资源访问的同步机制，即用于在并发控制中保证对互斥要求的满足。



**opengauss各类锁的定义**

```c++
#define NoLock 0  /*NoLock 不是锁模式，而是一个标志值，意思是“不要锁”*/

#define AccessShareLock 1  /* SELECT */
#define RowShareLock 2     /* SELECT FOR UPDATE/FOR SHARE */
#define RowExclusiveLock 3 /* INSERT, UPDATE, DELETE */
#define ShareUpdateExclusiveLock                         \
    4               /* VACUUM (non-FULL),ANALYZE, CREATE \
                     * INDEX CONCURRENTLY */
#define ShareLock 5 /* CREATE INDEX (WITHOUT CONCURRENTLY) */
#define ShareRowExclusiveLock                \
    6 /* like EXCLUSIVE MODE, but allows ROW \
       * SHARE */
#define ExclusiveLock                  \
    7 /* blocks ROW SHARE/SELECT...FOR \
       * UPDATE */
#define AccessExclusiveLock              \
    8 /* ALTER TABLE, DROP TABLE, VACUUM \
       * FULL, and unqualified LOCK TABLE */
```



**Lock结构体定义**

```c++
typedef struct LOCK {
    /* 哈希值 */
    LOCKTAG tag; /* 可锁对象的唯一标识符 */

    /* 数据 */
    LOCKMASK grantMask;           /* 已授予锁定类型的位掩码 */
    LOCKMASK waitMask;            /* 等待锁定类型的位掩码 */
    SHM_QUEUE procLocks;          /* PROCLOCK 对象关联列表 */
    PROC_QUEUE waitProcs;         /* 等待锁定的 PGPROC 对象列表 */
    int requested[MAX_LOCKMODES]; /* 请求的锁数 */
    int nRequested;               /* 请求的数组总数y */
    int granted[MAX_LOCKMODES];   /* 授予锁的总数 */
    int nGranted;                 /* 授予数组的总数 */
} LOCK;
```



**lockTagType的定义**

LOCKTAG 是在锁哈希表中查找 LOCK 项所需的关键信息。 LOCKTAG 值唯一标识可锁定对象。 LockTagType 枚举定义了我们可以锁定的不同类型的对象。我们最多可以处理 256 种不同的 LockTagType。

```c++
typedef enum LockTagType {
    LOCKTAG_RELATION, /* 整体关系 */
    /* 关系的ID是 DB OID加上REL OID,如果共享，则 DB OID = 0*/
    LOCKTAG_RELATION_EXTEND, /* the right to extend a relation */
    /* 与RELATION相同的ID */
    LOCKTAG_PARTITION,          
    LOCKTAG_PARTITION_SEQUENCE, /*分割序列*/
    LOCKTAG_PAGE,               /* one page of a relation */
    /* 页面ID为RELATION信息加上BlockNumber */
    LOCKTAG_TUPLE, /* 一个物理元组 */
    /* 元组的ID是PAGE info加上OffsetNumber */
    LOCKTAG_TRANSACTION, /* 等待完成的事务 */
    /* 事务的ID是它的TransactionId */
    LOCKTAG_VIRTUALTRANSACTION, /* virtual transaction (ditto) */
    /* 虚拟事务的ID是它的VirtualTransactionId */
    LOCKTAG_OBJECT, /* 非关系数据库对象 */
    /* 对象的ID是DB OID + CLASS OID + object OID + SUBID */
    LOCKTAG_CSTORE_FREESPACE, /* cstore可用空间 */

    /*
     *对象ID与pg_depend和pg_description具有相同的表示，但请注意我们将SUBID限制为16位。
     此外，我们对表空间等共享对象使用 DB OID = 0。
     */
    LOCKTAG_USERLOCK, /* 为旧的贡献锁的代码保留 */
    LOCKTAG_ADVISORY, /* 咨询用户锁 */
    LOCK_EVENT_NUM
} LockTagType;
```



**按作用域分类，opengauss数据库中的锁可分为三类：表级锁、行级锁、页级锁。**



**表级锁**

锁定粒度最大的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。其锁定粒度最大，触发锁冲突的概率最高，并发度最低。

**行级锁**

粒度最小的一种锁，只针对当前操作的行进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。

**页级锁**

页级锁是锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。因此，采取了折衷的页级锁，一次锁定相邻的一组记录。







