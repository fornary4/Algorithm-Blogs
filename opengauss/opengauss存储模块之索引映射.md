## opengauss存储模块之索引映射

**概述：buf_table.cpp的作用是将BufferTags映射到缓冲区索引**

这个文件中的在执行过程中没有自己的锁，调用者必须持有适当的BufMappingLock上的适当的锁，如注释中指定的锁。不能在这些函数中进行锁定，因为在大多数情况下，调用者需要调整缓冲区报头内容在锁被释放之前。

**BufferLookupEnt结构体的定义**

BufferLookupEnt缓冲区查找哈希表的入口

```c++
typedef struct {
    BufferTag key; /* 磁盘页的标签 */
    int id;        /* 关联的bufferID */
} BufferLookupEnt;
```

**重要函数定义**

**BufTableShmemSize:评估哈希表需要的空间**

```c++
Size BufTableShmemSize(int size)
{
    //调用dynahash定义的函数来估算空间
    return hash_estimate_size(size, sizeof(BufferLookupEnt));
}
```

**InitBufTable：初始化共享内存的哈希表来映射缓存**

参数size是所需的哈希表大小

```c++
void InitBufTable(int size)
{
    HASHCTL info;

    /* 假定这里不需要其他锁
     *
     * 需要映射的buffertag
     */
    info.keysize = sizeof(BufferTag);
    info.entrysize = sizeof(BufferLookupEnt);
    info.hash = tag_hash;
    info.num_partitions = NUM_BUFFER_PARTITIONS;

    t_thrd.storage_cxt.SharedBufHash = ShmemInitHash("Shared Buffer Lookup Table", size, size, &info,
                                                     HASH_ELEM | HASH_FUNCTION | HASH_PARTITION);
}
```

**BufTableLookup：查找给定的BufferTag;返回缓冲区ID，如果没有找到则返回-1**

在使用这个函数的时候，调用者必须至少持有标记分区的BufMappingLock上的共享锁

```c++
int BufTableLookup(BufferTag *tag, uint32 hashcode)//根据BufferTag和hashcode进行查找
{
    BufferLookupEnt *result = NULL;

    result = (BufferLookupEnt *)buf_hash_operate<HASH_FIND>(t_thrd.storage_cxt.SharedBufHash, tag, hashcode, NULL);

    if (SECUREC_UNLIKELY(result == NULL)) { //未找到的情况
        return -1;
    }

    return result->id;
}
```

**BufTableInsert：为给定的标签和缓冲区ID插入一个哈希表项，除非已经存在该标签的项**

成功插入时返回-1。如果存在冲突项，返回该条目中的缓冲区ID

调用者必须为标记的分区持有BufMappingLock上的排他锁

```c++
int BufTableInsert(BufferTag *tag, uint32 hashcode, int buf_id)
{
    BufferLookupEnt *result = NULL;
    bool found = false;

    Assert(buf_id >= 0);            /* -1代表插入成功 */
    Assert(tag->blockNum != P_NEW); /* 无效的标签 */

    result = (BufferLookupEnt *)buf_hash_operate<HASH_ENTER>(t_thrd.storage_cxt.SharedBufHash, tag, hashcode, &found);

    if (found) { /* 这个key已经存在 */
        return result->id;
    }

    result->id = buf_id;

    return -1;
}
```

**BufTableDelete：删除给定标签的hashtable条目(必须存在)**

调用者必须为标记的分区持有BufMappingLock上的排他锁

```c++
void BufTableDelete(BufferTag *tag, uint32 hashcode)
{
    BufferLookupEnt *result = NULL;
	
    result = (BufferLookupEnt *)buf_hash_operate<HASH_REMOVE>(t_thrd.storage_cxt.SharedBufHash, tag, hashcode, NULL);  //尝试移除entry

    if (result == NULL) { /* 不应该出现的情况 */
        ereport(ERROR, (errcode(ERRCODE_DATA_CORRUPTED), (errmsg("shared buffer hash table corrupted."))));
    }
}
```

