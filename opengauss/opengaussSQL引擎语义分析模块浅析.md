## opengaussSQL引擎语义分析模块浅析

### 实现引入一个概念：抽象语法树

抽象语法树（abstract syntax tree，AST）是源代码的抽象语法结构的树状表示，树上的每个节点都表示源代码中的一种结构，这所以说是抽象的，是因为抽象语法树并不会表示出真实语法出现的每一个细节，比如说，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现。抽象语法树并不依赖于源语言的语法，也就是说语法分析阶段所采用的上下文无文文法，因为在写文法时，经常会对文法进行等价的转换（消除左递归，回溯，二义性等）

<font color=red>语义分析就是对语法树（AST）进行有效性检查，检查语法树中对应的表、列、函数、表达式是否有对应的元数据，将抽象语法树转换为逻辑执行计划（关系代数表达式）。</font>

以下面这条sql语句为例

```sql
select salary from employee where id = 1;
```

可以划分的关键字、标识符、操作符、常量等原子单位，如下表所示。

| 词性   | 内容               |
| ------ | ------------------ |
| 关键词 | select,from,where  |
| 标识符 | salary,employee,id |
| 操作符 | =                  |
| 常量   | 1                  |

**抽象语法树如下**

![image-20210831231613281](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210831231613281.png)

### opengauss的语义分析流程

对于可优化语句，在每个被引用的表上获取一个合适的锁，后端的其他模块根据结果保留或重新获取这些锁。因此，可以对这些语句进行重要的语义分析。对于实用程序命令，这里没有获得锁（如果有，我们不能确定我们在执行时是否仍然拥有它们）。因此，实用程序命令的一般规则是将它们转储到未转换的查询节点中。 DECLARE CURSOR、EXPLAIN 和 CREATE TABLE AS 是例外，因为它们包含应该转换的可优化语句。

### 重要函数定义

**SQL查询query结构体定义**

解析分析是将所有语句转换为查询树，以便重写器和规划器进一步处理。实用程序语句（即不可优化语句）设置了utilityStmt 字段，而查询本身大多是虚拟的。 DECLARE CURSOR 是一个特例：它表示为一个 SELECT，但原始的 DeclareCursorStmt 存储在utilityStmt 中。 Planning 将 Query 树转换为以 PlannedStmt 节点为首的 Plan 树——Query 结构不被执行器使用。

```c++
typedef struct Query {
    NodeTag type;

    CmdType commandType;

    QuerySource querySource; /* 查询来源 */

    uint64 queryId; /* 查询标识符(可以通过插件设置) */

    bool canSetTag; 

    Node* utilityStmt; 


    bool hasAggs;         
    bool hasWindowFuncs; 
    bool hasSubLinks;     
    bool hasDistinctOn;  
    bool hasRecursive;    
    bool hasModifyingCTE; 
    bool hasForUpdate;   
    bool hasRowSecurity;  
    bool hasSynonyms;     

    List* cteList; 

    List* rtable;       
    FromExpr* jointree; 

    List* targetList; 

    List* starStart; 

    List* starEnd;

    List* starOnly; 

    List* returningList; 

    List* groupClause; 

    List* groupingSets; 

    Node* havingQual; 

    List* windowClause; 

    List* distinctClause; 
    
    List* sortClause; 

    Node* limitOffset; 
    Node* limitCount;

    List* rowMarks; 

    Node* setOperations; 

    List *constraintDeps; 
    HintState* hintState;
    
    ParamListInfo boundParamsQ;

    int mergeTarget_relation;
    List* mergeSourceTargetList;
    List* mergeActionList; 
    Query* upsertQuery;    
    UpsertExpr* upsertClause; 

    bool isRowTriggerShippable; 
    bool use_star_targets;     

    bool is_from_full_join_rewrite; 
                                    
    uint64 uniqueSQLId;             /* 由唯一SQL id使用 */
    bool can_push;
    bool        unique_check;               
    Oid* fixed_paramTypes; /* 对于 plpy CTAS 查询。 CTAS 是一个递归调用。CREATE 查询是第一个重写的。第 2 个重写的查询是 INSERT SELECT。没有这个属性，当分析 INSERT SELECT 查询时，数据库将有一个不知道的错误。 */
    int fixed_numParams;
} Query;
```

**语法解析函数parse_analyze**

parse_analyze 分析原始解析树并将其转换为查询形式。或者，可以提供有关 n 个参数类型的信息。不允许引用未由 paramTypes[] 定义的 n 个索引。结果是一个查询节点。可优化语句需要相当大的转换，而实用程序类型的语句只是挂在一个虚拟的 CMD_UTILITY 查询节点上。

```c++
Query* parse_analyze(
    Node* parseTree, const char* sourceText, Oid* paramTypes, int numParams, bool isFirstNode, bool isCreateView)
{
    ParseState* pstate = make_parsestate(NULL);
    Query* query = NULL;

    
    AssertEreport(sourceText != NULL, MOD_OPT, "para cannot be NULL");

    pstate->p_sourcetext = sourceText;

    if (numParams > 0) {
        parse_fixed_parameters(pstate, paramTypes, numParams);
    }

    query = transformTopLevelStmt(pstate, parseTree, isFirstNode, isCreateView);

    /* 处理插件hook是不安全的，因为动态库可能会被释放 */
    if (post_parse_analyze_hook && !(g_instance.status > NoShutdown)) {
        (*post_parse_analyze_hook)(pstate, query);
    }

    pfree_ext(pstate->p_ref_hook_state);
    free_parsestate(pstate);

    query->fixed_paramTypes = paramTypes;
    query->fixed_numParams = numParams;

    return query;
}
```

**可变参数语法解析函数parse_analyze_varparams**

当可以从上下文推断出有关 n 个符号数据类型的信息时，使用此变体。传入的 paramTypes[] 数组可以修改或扩大（通过 repalloc）。

```c++
Query* parse_analyze_varparams(Node* parseTree, const char* sourceText, Oid** paramTypes, int* numParams,
    char** paramTypeNames)
#endif
{
    ParseState* pstate = make_parsestate(NULL);
    Query* query = NULL;

  
    AssertEreport(sourceText != NULL, MOD_OPT, "para cannot be NULL");

    pstate->p_sourcetext = sourceText;
#ifdef ENABLE_MULTIPLE_NODES	
    parse_variable_parameters(pstate, paramTypes, numParams);
#else
    parse_variable_parameters(pstate, paramTypes, numParams, paramTypeNames);	
#endif	

    query = transformTopLevelStmt(pstate, parseTree);

    /* 确保参数类型一切正常 */
    check_variable_parameters(pstate, query);

    /* 处理插件hook是不安全的，因为动态库可能会被释放 */
    if (post_parse_analyze_hook && !(g_instance.status > NoShutdown)) {
        (*post_parse_analyze_hook)(pstate, query);
    }

    pfree_ext(pstate->p_ref_hook_state);
    free_parsestate(pstate);

    return query;
}
```

**子句分析函数parse_sub_analyze**

parse_sub_analyze 是递归分析子语句的入口

```c++
Query* parse_sub_analyze(Node* parseTree, ParseState* parentParseState, CommonTableExpr* parentCTE,
    bool locked_from_parent, bool resolve_unknowns)
{
    ParseState* pstate = make_parsestate(parentParseState);
    Query* query = NULL;

    pstate->p_parent_cte = parentCTE;
    pstate->p_locked_from_parent = locked_from_parent;
    pstate->p_resolve_unknowns = resolve_unknowns;
    if (u_sess->attr.attr_sql.td_compatible_truncation && u_sess->attr.attr_sql.sql_compatibility == C_FORMAT)
        set_subquery_is_under_insert(pstate); /* 设置p_is_in_insert用于解析状态 */

    query = transformStmt(pstate, parseTree);

    free_parsestate(pstate);

    return query;//返回分析后的查询语句
}
```

