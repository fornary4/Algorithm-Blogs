## opengauss存储模块之空闲空间管理

**opengauss存储流程图**

![image-20210905222216552](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210905222216552.png)

**freespace.cpp位于存储模块freespace目录下，作用是负责空间映射，查找关联的空闲空间**

可用空间映射需要跟踪页面上的可用空间量，快速搜索具有足够可用空间的页面。 FSM 存储在所有堆关系和那些需要它的索引访问方法的专用关系分支中

**实现逻辑分析**

- FSMAddress记录了页面地址，以方便逻辑寻址
- 通过构造查找树来搜索可用空间
- 主要工作流程为搜索查找树、将可用空间量转换为FSM类型、地址映射

**代码分析**

**FSMAddress结构体的定义**

内部 FSM 按照逻辑寻址方案工作，搜索树的每一层都可以被认为是一个单独的可寻址文件

```c++
typedef struct {
    int level;     /* 层号 */
    int logpageno; /* 页面编号 */
} FSMAddress;
```

**函数RecordAndGetPageWithFreeSpace**

更新有关页面的信息，并刷新状态。与单独的 RecordPageWithFreeSpace + GetPageWithFreeSpace 调用相比，这种组合形式可以节省时间和空间开销

```c++
lockNumber RecordAndGetPageWithFreeSpace(Relation rel, BlockNumber oldPage, Size oldSpaceAvail, Size spaceNeeded)
{
    int old_cat = fsm_space_avail_to_cat(oldSpaceAvail);
    int search_cat = fsm_space_needed_to_cat(spaceNeeded);
    FSMAddress addr;
    uint16 slot;
    int search_slot;

    /* 获取代表堆块的FSM字节的位置 */
    addr = fsm_get_location(oldPage, &slot);

    search_slot = fsm_set_and_search(rel, addr, slot, (uint8)old_cat, (uint8)search_cat);
    /*
     * 如果 fsm_set_and_search 找到合适的新块，则返回该块。否则，照常搜索。
     */
    if (search_slot != -1)
        return fsm_get_heap_blk(addr, (uint16)search_slot);
    else
        return fsm_search(rel, (uint8)search_cat);
}

```

**函数UpdateFreeSpaceMap**

空闲空间的映射从上层一直到根，以确保不会丢失对刚刚插入的新块的跟踪。这旨在在向关系添加许多新块后使用，每次单个页面的数据更改时都更新树的上层是不值得的，但对于批量扩展来说这是值得的

```c++
void UpdateFreeSpaceMap(Relation rel, BlockNumber startBlkNum, BlockNumber endBlkNum, Size freespace)
{
    int new_cat = fsm_space_avail_to_cat(freespace);
    FSMAddress addr;
    uint16 slot;
    BlockNumber blockNum;
    BlockNumber lastBlkOnPage;

    blockNum = startBlkNum;

    while (blockNum <= endBlkNum) {
        /*
         * 找到这个区块的 FSM 地址；一直更新树到根
         */
        addr = fsm_get_location(blockNum, &slot);
        fsm_update_recursive(rel, addr, (uint8)new_cat);

        /*
         * 获取此 FSM 页面上的最后一个区块编号。如果它大于或等于我们的endBlkNum，我们就完成了。
         * 否则，前进到下一页的第一个块。
         */
        lastBlkOnPage = fsm_get_lastblckno(rel, addr);
        if (lastBlkOnPage >= endBlkNum)
            break;
        blockNum = lastBlkOnPage + 1;
    }
}
```

**函数fsm_space_needed_to_cat**

判断页面需要具有什么类别才能容纳 x 字节的数据，虽然 fsm_size_to_avail_cat() 向下取整，但在这里需要向上取整。

```c++
static uint8 fsm_space_needed_to_cat(Size needed)
{
    int cat;

    /* 不能要求比最高类别更大的空间 */
    if (needed > MaxFSMRequestSize) {
        ereport(
            ERROR, (errcode(ERRCODE_DATATYPE_MISMATCH), errmsg("invalid FSM request size %lu", (unsigned long)needed)));
    }

    if (needed == 0) {
        return 1;
    }

    cat = (needed + FSM_CAT_STEP - 1) / FSM_CAT_STEP;
	
    /* 为Byte时的情况 */
    if (cat > 255) {
        cat = 255;
    }

    return (uint8)cat;
}
```

**函数fsm_readbuf**

读取一个fsm页面，如果页面不存在，则返回 InvalidBuffer，或者如果extend为真，则扩展 FSM 文件

```c++
static Buffer fsm_readbuf(Relation rel, const FSMAddress& addr, bool extend)
{
    BlockNumber blkno = fsm_logical_to_physical(addr);
    Buffer buf;

    RelationOpenSmgr(rel);

    /*
     * 如果还没有得到缓存FSM的大小，先要检查它，还要重新检查请求的块是否已经结束，
     * 因为我们的缓存值可能已经过时。
     */
    if (rel->rd_smgr->smgr_fsm_nblocks == InvalidBlockNumber || blkno >= rel->rd_smgr->smgr_fsm_nblocks) {
        if (smgrexists(rel->rd_smgr, FSM_FORKNUM))
            rel->rd_smgr->smgr_fsm_nblocks = smgrnblocks(rel->rd_smgr, FSM_FORKNUM);
        else
            rel->rd_smgr->smgr_fsm_nblocks = 0;
    }

    /* 处理EOF以外的请求 */
    if (blkno >= rel->rd_smgr->smgr_fsm_nblocks) {
        if (extend)
            fsm_extend(rel, blkno + 1);
        else
            return InvalidBuffer;
    }

    /*
     * 使用ZERO_ON_ERROR模式，并在必要时初始化页面。
     * FSM信息可能并不准确，因此最好清除损坏的页面。
     */
    buf = ReadBufferExtended(rel, FSM_FORKNUM, blkno, RBM_ZERO_ON_ERROR, NULL);
    if (PageIsNew(BufferGetPage(buf))) {
        LockBuffer(buf, BUFFER_LOCK_EXCLUSIVE);
        if (PageIsNew(BufferGetPage(buf))) {
            PageInit(BufferGetPage(buf), BLCKSZ, 0);
        }
        LockBuffer(buf, BUFFER_LOCK_UNLOCK);
    }

    return buf;
}
```

