## SQL引擎查询解析模块之数据类型处理（一）

**概述：parse_type.cpp的作用是处理查询语句的数据类型**

**HeapTupleData结构体的定义**

`HeapTupleData 是指向元组的内存数据结构`

HeapTupleData的使用方法

- 指向磁盘缓冲区中元组的指针：t_data 直接指向缓冲区（代码最好保持一个引脚，但这不会反映在 HeapTupleData 本身中）。
- 无指针：t_data 为 NULL。这在某些功能中用作故障指示。
- palloc'd 元组的一部分：HeapTupleData 本身和元组形成一个单独的 palloc'd 块。 t_data 指向紧跟在 HeapTupleData 结构之后的内存位置（偏移量 HEAPTUPLESIZE）。这是 heap_form_tuple 和相关例程的输出格式。
- 单独分配的元组：t_data 指向一个与 HeapTupleData 不相邻的 palloc 块。 （这种情况已被弃用，因为它很难与情况 1 区分开来。它应该仅在代码知道情况 1 永远不会适用的有限上下文中使用。）
- 单独分配的最小元组： t_data 指向 MinimalTuple 开始之前的 MINIMAL_TUPLE_OFFSET 字节.与前一个案例一样，这不能通过检查与案例 1 分开；设置或销毁此表示的代码必须知道它在做什么。 t_len 应该始终有效，除非在指针指向空的情况下。如果 HeapTupleData 指向磁盘缓冲区，或者它代表磁盘上元组的副本，则 t_self 和 t_tableOid 应该是有效的。它们应该在制造的元组中明确设置为无效。



```c++
typedef struct HeapTupleData {
    uint32 t_len;           /* 数据长度 */
    uint1 tupTableType = HEAP_TUPLE;
    int2   t_bucketId;
    ItemPointerData t_self; /* 指向自身的迭代指针 */
    Oid t_tableOid;         /* 元组来源表 */
    TransactionId t_xid_base;
    TransactionId t_multi_base;
#ifdef PGXC
    uint32 t_xc_node_id; /* 元组来自的数据节点 */
#endif
    HeapTupleHeader t_data; /* 元组数据 */
} HeapTupleData;
```

**LookupTypeNameExtended函数分析**

功能：给定一个 TypeName 对象，查找该类型的 pg_type syscache 条目。如果找不到这样的类型，则返回 NULL。如果找到类型，则计算 TypeName 结构中表示的 typmod 值并将其存储到 typmod_p 中。

```c++
Type LookupTypeNameExtended(ParseState* pstate, const TypeName* typname, int32* typmod_p, bool temp_ok,
                            bool print_notice)
{
    Oid typoid;
    HeapTuple tup;
    int32 typmod;

    if (typname->names == NIL) {
        /* 我们已经有了OID，如果它是一个内部生成的TypeName */
        typoid = typname->typeOid;
    } else if (typname->pct_type) {
        /* 处理对现有字段类型的%TYPE引用*/
        RangeVar* rel = makeRangeVar(NULL, NULL, typname->location);
        char* field = NULL;
        Oid relid;
        AttrNumber attnum;

        /* ：解析姓名列表*/
        switch (list_length(typname->names)) {
            case 1:
                ereport(ERROR,
                    (errcode(ERRCODE_SYNTAX_ERROR),
                        errmsg(
                            "improper %%TYPE reference (too few dotted names): %s", NameListToString(typname->names)),
                        parser_errposition(pstate, typname->location)));
                break;
            case 2:
                rel->relname = strVal(linitial(typname->names));
                field = strVal(lsecond(typname->names));
                break;
            case 3:
                rel->schemaname = strVal(linitial(typname->names));
                rel->relname = strVal(lsecond(typname->names));
                field = strVal(lthird(typname->names));
                break;
            case 4:
                rel->catalogname = strVal(linitial(typname->names));
                rel->schemaname = strVal(lsecond(typname->names));
                rel->relname = strVal(lthird(typname->names));
                field = strVal(lfourth(typname->names));
                break;
            default:
                ereport(ERROR,
                    (errcode(ERRCODE_SYNTAX_ERROR),
                        errmsg(
                            "improper %%TYPE reference (too many dotted names): %s", NameListToString(typname->names)),
                        parser_errposition(pstate, typname->location)));
                break;
        }

        relid = RangeVarGetRelid(rel, NoLock, false);
        attnum = get_attnum(relid, field);
        if (attnum == InvalidAttrNumber) {
            ereport(ERROR,
                (errcode(ERRCODE_UNDEFINED_COLUMN),
                    errmsg("column \"%s\" of relation \"%s\" does not exist", field, rel->relname),
                    parser_errposition(pstate, typname->location)));
        }
        typoid = get_atttype(relid, attnum);

        /* 这个构造不应该有数组指示符 */
        Assert(typname->arrayBounds == NIL);

        if (print_notice) {
            ereport(NOTICE,
                (errmsg("type reference %s converted to %s", TypeNameToString(typname), format_type_be(typoid))));
        }
    } else {
        /* 对类型名的普通引用 */
        char* schemaname = NULL;
        char* typeName = NULL;

        /* 解析姓名列表 */
        DeconstructQualifiedName(typname->names, &schemaname, &typeName);

        if (schemaname != NULL) {
            /* 只查看特定的模式 */
            Oid namespaceId;

            namespaceId = LookupExplicitNamespace(schemaname);
            typoid = GetSysCacheOid2(TYPENAMENSP, PointerGetDatum(typeName), ObjectIdGetDatum(namespaceId));
        } else {
            /* 非限定类型名，因此搜索搜索路径 */
            typoid = TypenameGetTypidExtended(typeName, temp_ok);
        }

        /* 如果是数组引用，则返回数组类型 */
        if (typname->arrayBounds != NIL) {
            typoid = get_array_type(typoid);
        }
    }

    if (!OidIsValid(typoid)) {
        if (typmod_p != NULL) {
            *typmod_p = -1;
        }
        return NULL;
    }

    /* 不支持黑名单中的类型 */
    if (!u_sess->attr.attr_common.IsInplaceUpgrade && IsTypeInBlacklist(typoid)) {
        ereport(ERROR,
            (errcode(ERRCODE_FEATURE_NOT_SUPPORTED), errmsg("type %s is not yet supported.", format_type_be(typoid))));
    }

    tup = SearchSysCache1(TYPEOID, ObjectIdGetDatum(typoid));

    /* 不应该出现下面的情形 */
    if (!HeapTupleIsValid(tup)) {
        ereport(ERROR, (errcode(ERRCODE_CACHE_LOOKUP_FAILED), errmsg("cache lookup failed for type %u", typoid)));
    }

    typmod = typenameTypeMod(pstate, typname, (Type)tup);

    if (typmod_p != NULL) {
        *typmod_p = typmod;
    }

    return (Type)tup;
}
```



**typenameType函数分析**

typenameType—给定一个TypeName，返回一个Type结构和typmod。这与 LookupTypeName 等效，只是如果找不到或未定义类型，则会报告合适的错误消息。因此，它的调用者可以假设结果是一个完全有效的类型。

```c++
Type typenameType(ParseState* pstate, const TypeName* typname, int32* typmod_p)
{
    Type tup;

    tup = LookupTypeName(pstate, typname, typmod_p);

    /*
     *如果类型是relation，那么我们检查表是否在安装组中
     */
    if (!in_logic_cluster() && !IsTypeTableInInstallationGroup(tup)) {
        ereport(ERROR,
            (errcode(ERRCODE_FEATURE_NOT_SUPPORTED),
                errmsg("type '%s' must be in installation group", TypeNameToString(typname))));
    }

    if (tup == NULL) {
        ereport(ERROR,
            (errcode(ERRCODE_UNDEFINED_OBJECT),
                errmsg("type \"%s\" does not exist", TypeNameToString(typname)),
                parser_errposition(pstate, typname->location)));
    }
        
    if (!((Form_pg_type)GETSTRUCT(tup))->typisdefined) {
        ereport(ERROR,
            (errcode(ERRCODE_UNDEFINED_OBJECT),
                errmsg("type \"%s\" is only a shell", TypeNameToString(typname)),
                parser_errposition(pstate, typname->location)));
    }
    return tup;
}
```

