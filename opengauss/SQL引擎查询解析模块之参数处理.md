## SQL引擎查询解析模块之参数处理

**概述：**

parse_param.cpp的作用是处理sql查询语句里的参数。参数处理有以下两种情况： 

- 具有已知类型的固定参数列表 
- 可扩展的参数列表，即可变参数，其类型可以选择从上下文中确定 

在这两种情况下，仅支持显式引用（ParamRef 节点）。

**实现机制分析**

- 定义了FixedParamState和VarParamState，分别对应固定参数和可变参数

- 定义了函数parse_fixed_parameters和parse_variable_parameters，分别对应上述两种情况

- 
  check_parameter_resolution_walker通过遍历生成的语法树来匹配参数符号
  

**代码分析**

**参数Param结构体的定义**

```c++
typedef struct Param {
    Expr xpr;
    ParamKind paramkind; /* 参数类型 */
    int paramid;         /* 参数的数字ID */
    Oid paramtype;       /* 参数数据类型的pg_type OID */
    int32 paramtypmod;   /* Typmod值 */
    Oid paramcollid;     /* 排序的OID */
    int location;        /* token位置  */
} Param;
```

**FixedParamState和VarParamState结构体的定义**

在 varparams 情况下，调用者提供的 OID 数组可以根据需要重新分配更大的空间。零数组条目表示尚未看到参数编号，而 UNKNOWNOID 表示已使用该参数但其类型尚不清楚。

```c++
typedef struct FixedParamState {
    Oid* paramTypes; /* 参数类型oid的数组 */
    int numParams;   /* 参数项数 */
} FixedParamState;

typedef struct VarParamState {
    Oid** paramTypes; /* 参数类型oid的数组 */
    int* numParams;   /* 参数项数 */
#ifndef ENABLE_MULTIPLE_NODES	
    char **paramTypeNames;
#endif	
} VarParamState;
```

**固定参数的处理**

创建一个FixedParamState对象并初始化参数

```c++
void parse_fixed_parameters(ParseState* pstate, Oid* paramTypes, int numParams)
{
    FixedParamState* parstate = (FixedParamState*)palloc(sizeof(FixedParamState));

    parstate->paramTypes = paramTypes;
    parstate->numParams = numParams;
    pstate->p_ref_hook_state = (void*)parstate;
    pstate->p_paramref_hook = fixed_paramref_hook;
    /* 不需要使用p_coerce_param_hook */
}
```

**可变参数的处理**

设置为处理包含对可变参数的引用的查询

```c++
void parse_variable_parameters(ParseState* pstate, Oid** paramTypes, int* numParams,
     char** paramTypeNames)
{
    VarParamState* parstate = (VarParamState*)palloc(sizeof(VarParamState));

    parstate->paramTypes = paramTypes;
    parstate->numParams = numParams;
#ifndef ENABLE_MULTIPLE_NODES
    parstate->paramTypeNames = paramTypeNames;
    pstate->p_post_columnref_hook = variable_post_column_ref_hook;
#endif
    pstate->p_ref_hook_state = (void*)parstate;
    pstate->p_paramref_hook = variable_paramref_hook;
    pstate->p_coerce_param_hook = variable_coerce_param_hook;
}
```

**variable_coerce_param_hook**

将参数强制转换为查询请求的数据类型

```c++
static Node* variable_coerce_param_hook(
    ParseState* pstate, Param* param, Oid targetTypeId, int32 targetTypeMod, int location)
{
    if (param->paramkind == PARAM_EXTERN && param->paramtype == UNKNOWNOID) {
        /*
         * 输入是一个先前未确定类型的Param
         */
        VarParamState* parstate = (VarParamState*)pstate->p_ref_hook_state;
        Oid* paramTypes = *parstate->paramTypes;
        int paramno = param->paramid;

        if (paramno <= 0 || 
            paramno > *parstate->numParams) {
            ereport(ERROR,
                (errcode(ERRCODE_UNDEFINED_PARAMETER),
                    errmsg("there is no parameter $%d", paramno),
                    parser_errposition(pstate, param->location)));
        }
        if (paramTypes[paramno - 1] == UNKNOWNOID) {
            /* 已经成功地解析了类型 */
            paramTypes[paramno - 1] = targetTypeId;
        } else if (paramTypes[paramno - 1] == targetTypeId) {
            /* 之前解析了此类型，并且匹配 */
        } else {

            ereport(ERROR,
                (errcode(ERRCODE_AMBIGUOUS_PARAMETER),
                    errmsg("inconsistent types deduced for parameter $%d", paramno),
                    errdetail("%s versus %s", format_type_be(paramTypes[paramno - 1]), format_type_be(targetTypeId)),
                    parser_errposition(pstate, param->location)));
        }

        param->paramtype = targetTypeId;

        param->paramtypmod = -1;

        /*
         * 此模块始终将 Param 的排序规则设置为其数据类型的默认值
         */
        param->paramcollid = get_typcollation(param->paramtype);

        /* 使用最左边的参数和强制的位置 */
        if (location >= 0 && (param->location < 0 || location < param->location)) {
            param->location = location;
        }
        return (Node*)param;
    }

    /* Else信号继续正常强制 */
    return NULL;
}

```

