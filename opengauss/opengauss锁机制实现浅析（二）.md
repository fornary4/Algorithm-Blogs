## opengauss锁机制实现浅析（二）



**表级锁模式解读**

| 锁模式       | 解释                                                    |
| ------------------------- | -------------------------------- |
| Access Share | 只与Access Exclusive锁模式冲突。查询命令（Select command）将会在它查询的表上获取Access Shared锁，一般地，任何一个对表上的只读查询操作都将获取这种类型锁。 |
| Row Share | 与Exclusive和Access Exclusive锁模式冲突。Select for update和Select for share命令将获得这种类型锁，并且所有被引用但没有for update 的表上会加上Access Shared锁。 |
| Row Exclusive | 与Share，Shared Row Exclusive，Exclusive，Access Exclusive模式冲突。Update/Delete/Insert命令会在目标表上获得这种类型的锁，并且在其它被引用的表上加上Access Share锁，一般地，更改表数据的命令都将在这张表上获得Row Exclusive锁。 |
| Share Update Exclusive | Share Update Exclusive，Share，Share Row Exclusive，Exclusive，Access exclusive模式冲突，这种模式保护一张表不被并发的模式更改和Vacuum。Vacuum(without full)，Analyze 和 Create index concur-ently命令会获得这种类型锁。 |
| Share | 与Row Exclusive，Shared Update Exclusive，Share Row Exclusive，Exclusive，Access exclusive锁模式冲突，这种模式保护一张表数据不被并发的更改。Create index命令会获得这种锁模式。 |
| Share Row Exclusive | 与Row Exclusive，Share Update Exclusive，Shared，Shared Row Exclusive，Exclusive，Access Exclusive锁模式冲突。任何PostgreSQL命令不会自动获得这种类型的锁。 |
| Exclusive | 与ROW Share , Row Exclusive, Share  Update  Exclusive, Share , Share  Row Exclusive, Exclusive, Access Exclusive模式冲突，这种锁模式仅能与Access Share 模式并发。任何PostgreSQL命令不会自动获得这种类型的锁。 |
| Access Exclusive | 与所有模式锁冲突(Access Share，Row Share，Row Exclusive，Share  Update Exclusive，Share , Share Row Exclusive，Exclusive，Access Exclusive)，这种模式保证了当前只有一个人访问这张表；ALTER TABLE，DROP TABLE，TRUNCATE，REINDEX，CLUSTER，VACUUM FULL 命令会获得这种类型锁，在Lock table 命令中，如果没有申明其它模式，它也是默认模式。 |

**行级锁模式**

行级锁模式只有两种，分别是共享锁和排他锁，也就是读锁或写锁。由于多版本的实现，实际读取行数据时，并不会在行上执行任何锁。特定行上的行级锁是在行被更新的时候自动请求的（或者被删除时或标记为更新）。 锁一直保持到事务提交或者回滚。行级锁不影响对数据的查询；它们只阻塞对同一行的写入。 



**并发控制（ＭＶＣＣ）：**

==在一般情况下，读锁与写锁是不能并发的，为了提高效率，产生了MVCC技术，opengauss使用MVCC来实现并发控制==

**什么是MVCC**

MVCC，Multi-Version Concurrency Control，多版本并发控制。MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问；在编程语言中实现事务内存。

如果有人从数据库中读数据的同时，有另外的人写入数据，有可能读数据的人会看到半写或者不一致的数据。有很多种方法来解决这个问题，叫做并发控制方法。最简单的方法，通过加锁，让所有的读者等待写者工作完成，但是这样效率会很差。MVCC 使用了一种不同的手段，每个连接到数据库的读者，在某个瞬间看到的是数据库的一个快照，写者写操作造成的变化在写操作完成之前（或者数据库事务提交之前）对于其他的读者来说是不可见的。

**MVCC的实现原理**

当一个 MVCC 数据库需要更一个一条数据记录的时候，它不会直接用新数据覆盖旧数据，而是将旧数据标记为过时（obsolete）并在别处增加新版本的数据。这样就会有存储多个版本的数据，但是只有一个是最新的。这种方式允许读者读取在他读之前已经存在的数据，即使这些在读的过程中半路被别人修改、删除了，也对先前正在读的用户没有影响。这种多版本的方式避免了填充删除操作在内存和磁盘存储结构造成的空洞的开销，但是需要系统周期性整理（sweep through）以真实删除老的、过时的数据。对于面向文档的数据库（Document-oriented database，也即半结构化数据库）来说，这种方式允许系统将整个文档写到磁盘的一块连续区域上，当需要更新的时候，直接重写一个版本，而不是对文档的某些比特位、分片切除，或者维护一个链式的、非连续的数据库结构。

MVCC 提供了时点（point in time）一致性视图。MVCC 并发控制下的读事务一般使用时间戳或者事务 ID去标记当前读的数据库的状态（版本），读取这个版本的数据。读、写事务相互隔离，不需要加锁。读写并存的时候，写操作会根据目前数据库的状态，创建一个新版本，并发的读则依旧访问旧版本的数据。
