## SQL引擎查询解析模块之数据类型处理（二）

### 实现机制分析

- 在opengauss中用unsigned int表示oid（即object  id），用来区分不同的对象，在Oid cstoreSupportType[]数组中声明了数据库支持的所有类型
- 定义了TypeName、TypeID、TypeMod来标识不同的类型，所有的函数都是对这些量的操作来区分数据类型
- 在读取TypeName时，将表示 TypeName 名称的字符串附加到 StringInfo，再加到缓冲区，以提高读取效率

**TypeName结构体的定义**

<font color=red>TypeName 的作用是在定义中指定类型</font> ，对于内部生成的 TypeName 结构，通过 OID 指定类型通常比通过名称更容易。果“names”为NIL(空list)，则实际类型 OID 由 typeOid 给出，否则 typeOid 未使用。类似地，如果“typmods”是 NIL，那么实际的 typmod 应该在typemod 中预先指定，否则 typemod 未被使用。如果 pct_type 为 TRUE，则 names 实际上是一个字段名称，系统会查找该字段的类型。正常情况下，名称是一个类型名称，可能用模式和数据库名称限定。

```c++
typedef struct TypeName {
    NodeTag type;
    List *names;       /* 限定名(值字符串列表) */
    Oid typeOid;       /* 由OID识别的类型 */
    bool setof;        /* 是set */
    bool pct_type;     /*规定的类型 */
    List *typmods;     /* 类型修饰符表达式 */
    int32 typemod;     /* 指定类型修饰符 */
    List *arrayBounds; /* 数组下标界限 */
    int location;      /* token位置 */
} TypeName;

```

**处理流程分析**

通过typenameTypeMod、typenameTypeIdAndMod等函数获取对象的类型，在读取的时候将TypeName加到缓冲区，生成一个字符串来表示 TypeNames 列表的名称，按名称查找排序规则，并返回 OID，在给定 ColumnDef 节点和先前确定的列类型 OID 的情况下，获取要用于定义的列的排序规则，最后根据系统预先定义的数据类型来标识传入参数的数据类型。

### 重要函数分析

**IsTypeTableInInstallationGroup的作用是检查类型是否在安装组中**

通过（groupname != NULL && installation_groupname != NULL && strcmp(groupname, installation_groupname) != 0) 来判断是否在安装组中

```c++
bool IsTypeTableInInstallationGroup(const Type type_tup)
{
    if (type_tup && !IsInitdb && IS_PGXC_COORDINATOR) {
        Form_pg_type typeForm = (Form_pg_type)GETSTRUCT(type_tup);
        char* groupname = NULL;
        Oid groupoid = InvalidOid;

        if (OidIsValid(typeForm->typrelid)) {
            char relkind = get_rel_relkind(typeForm->typrelid);
            if (RELKIND_VIEW != relkind && RELKIND_CONTQUERY != relkind) {
                groupoid = ng_get_baserel_groupoid(typeForm->typrelid, relkind);
            }
			//检查Oid的合法性
            if (OidIsValid(groupoid)) {
                groupname = get_pgxc_groupname(groupoid);
            }

            char* installation_groupname = PgxcGroupGetInstallationGroup();
            //判定是否在安装组
            if (groupname != NULL && installation_groupname != NULL && strcmp(groupname, installation_groupname) != 0) {
                return false;
            }
        }
    }
    return true;
}
```

**IsTypeInBlacklist的作用是判断类型是否在黑名单中**

通过typoid来进行具体的判断

```c++
bool IsTypeInBlacklist(Oid typoid)
{
    bool isblack = false;

    switch (typoid) {
#ifdef ENABLE_MULTIPLE_NODES
        case XMLOID:
#endif /* 启用多节点 */
        case LINEOID:
        case PGNODETREEOID:
            isblack = true;
            break;
        default:
            break;
    }

    return isblack;
}
```



**IsTypeSupportedByORCRelation的作用是判断类型是否被支持**

函数的参数是typeOid，在Oid supportType[]定义了系统支持的类型，通过判断typeOid是否在数组中来判断类型是否被支持

```c++
bool IsTypeSupportedByORCRelation(_in_ Oid typeOid)
{
    /* 不支持用户定义类型 */
    if (typeOid >= FirstNormalObjectId) {
        return false;
    }

    static Oid supportType[] = {BOOLOID,
        OIDOID,
        INT8OID,
        INT2OID,
        INT4OID,
        INT1OID,
        NUMERICOID,
        CHAROID,
        BPCHAROID,
        VARCHAROID,
        NVARCHAR2OID,
        TEXTOID,
        CLOBOID,
        FLOAT4OID,
        FLOAT8OID,
        DATEOID,
        TIMESTAMPOID,
        INTERVALOID,
        TINTERVALOID,
        TIMESTAMPTZOID,
        TIMEOID,
        TIMETZOID,
        SMALLDATETIMEOID,
        CASHOID};
    if (DATEOID == typeOid && C_FORMAT == u_sess->attr.attr_sql.sql_compatibility) {
        ereport(ERROR,
            (errcode(ERRCODE_FEATURE_NOT_SUPPORTED),
                errmodule(MOD_HDFS),
                errmsg("Date type is unsupported for hdfs table in C-format database.")));
    }

    for (uint32 i = 0; i < sizeof(supportType) / sizeof(Oid); ++i) {
        if (supportType[i] == typeOid) {
            return true;
		}
    }
    return false;
}
```

