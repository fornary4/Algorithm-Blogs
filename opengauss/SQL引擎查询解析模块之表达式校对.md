## SQL引擎查询解析模块之表达式校对

**概述：parse_collate.cpp的作用是对解析后的sql语句进行校对，并分配整理信息的处理流程**

opengauss选择在表达式解析分析的输出的后传递中处理校对分析，这是因为处理需要比完成的语法树中更多的状态来执行。如果在构建语法树时即时进行，则所有这些状态都必须永久保存在表达式节点树中，这样，额外的存储只是这个递归例程中的局部变量。

完成后的语法树中实际保存的信息是： 

1. 每个表达式节点的输出排序规则，如果返回不可排序的数据类型，则为 InvalidOid。如果结果类型可整理但整理不确定，则这也可以是 InvalidOid。 
2. 执行每个函数时使用的排序规则。 InvalidOid 意味着没有可整理的输入或它们的整理是不确定的。此值仅存储在可能调用使用排序规则的函数的节点类型中。

**处理逻辑**

- 通过CollateStrength来判断表达式的规范性，并判断是否需要重新排序
- 具有不确定校对规则的情况可能会导致在运行时抛出错误。在处理时需要获取函数的整理信息，以便在解析时抛出这些错误
- 定义函数assign_aggregate_collations和assign_ordered_set_collations分别处理聚合集和有序集

**代码分析**

**CollateStrength的定义**

整理强度（SQL 标准称之为“派生”）。选择顺序以允许比较有效地工作

```c++
typedef enum {
    COLLATE_NONE,     /* 表达式具有不可排序的数据类型 */
    COLLATE_IMPLICIT, /* 排序规则是隐式派生的 */
    COLLATE_CONFLICT, /* 我们遇到了隐式排序的冲突 */
    COLLATE_EXPLICIT  /* 排序规则是显式派生的 */
} CollateStrength;
```

**函数定义**

```c++
static bool assign_query_collations_walker(Node* node, ParseState* pstate);
static bool assign_collations_walker(Node* node, assign_collations_context* context);
static void merge_collation_state(Oid collation, CollateStrength strength, int location, Oid collation2, int location2,
    assign_collations_context* context);
static void assign_aggregate_collations(Aggref* aggref, assign_collations_context* loccontext);
static void assign_ordered_set_collations(Aggref* aggref, assign_collations_context* loccontext);

```

**get_valid_collation**

根据类型设置适当的排序规则、强度和位置。

```c++
static void get_valid_collation(Oid& collation, CollateStrength& strength, int& location, Oid typcollation,
    bool collatable, int expr_location, assign_collations_context context)
{
    if (OidIsValid(typcollation)) {
        /* typlation(来自一个节点)是可整理的 */
        if (collatable) {
            /* 排序状态从子节点弹出 */
            collation = context.collation;
            strength = context.strength;
            location = context.location;
        } else {
            /*
             * 无需任何可整理输入即可生成可整理输出，
             * 使用类型的排序规则（通常是 DEFAULT_COLLATION_OID，但对于作用域可能不同）。
             */
            collation = typcollation;
            strength = COLLATE_IMPLICIT;
            location = expr_location;
        }
    } else {
        /* Node的结果类型是不可排序的 */
        collation = InvalidOid;
        strength = COLLATE_NONE;
        location = -1; /* 不会使用 */
    }
}
```

**assign_query_collations**

用排序信息标记给定查询中的所有表达式，在完成表达式的解析分析后，这应该应用于每个 Query

```c++
void assign_query_collations(ParseState* pstate, Query* query)
{
    /*
     * 我们只使用 query_tree_walker() 来访问所有包含的表达式。不过，我们可以跳过范围表和 CTE 子查询，
     * 因为 RTE 和子查询最好已经处理过
     */
    (void)query_tree_walker(query,
        (bool (*)())assign_query_collations_walker,
        (void*)pstate,
        QTW_IGNORE_RANGE_TABLE | QTW_IGNORE_CTE_SUBQUERIES);
}
```

**assign_query_collations_walker**

query_tree_walker 找到的每个表达式都是独立处理的。请注意，query_tree_walker 可能会传递一个完整的列表，例如目标列表，在这种情况下，每个子表达式都必须独立处理

```c++
static bool assign_query_collations_walker(Node* node, ParseState* pstate)
{
    /* 不需要为空子表达式做任何事情 */
    if (node == NULL) {
        return false;
    }
    /*
     * 不需要递归到一个集合操作树，它已经在transformSetOperationStmt中完全处理。
     */
    if (IsA(node, SetOperationStmt)) {
        return false;
    }
    if (IsA(node, List)) {
        assign_list_collations(pstate, (List*)node);
    } else {
        assign_expr_collations(pstate, node);
    }
    return false;
}
```
