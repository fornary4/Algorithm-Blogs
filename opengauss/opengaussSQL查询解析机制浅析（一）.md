## opengaussSQL查询解析机制浅析（一）

![image-20210830222755563](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210830222755563.png)

应用程序的SQL需要经过SQL解析生成逻辑执行计划、经过查询优化生成物理执行计划，然后将物理执行计划转交给查询执行引擎做物理算子的执行操作。

### SQL查询具体执行流程

1. **先查询高速缓存(library cache)**，检查Query语句是否完全匹配，接着再检查是否具有权限，都成功则直接取数据返回。服务器进程在接到客户端传送过来的 SQL 语句时，不会直接去数据库查询。而是会先在数据库的高速缓存中去查找，是否存在相同语句的执行计划。
2. **语句合法性检查(data dict cache)**。当在高速缓存中找不到对应的 SQL 语句时,则服务器进程就会开始检查这条语句的合法性。这里主要是对 SQL 语句的语法进行检查,看看其是否合乎语法规则。如果服务器进程认为这条 SQL 语句不符合语法规则的时候,就会把这个错误信息,反馈给客户端。在这个语法检查的过程中,不会对 SQL 语句中所包含的表名、列名等等进行 SQL 他只是语法上的检查。
3. **语言含义检查(data dict cache)**。若 SQL 语句符合语法上的定义的话,则服务器进程接下去会对语句中的字段、表等内容进行检查。看看这些字段、表是否在数据库中。如果表名与列名不准确的话,则数据库会就会反馈错误信息给客户端。
4. **获得对象解析锁(control structer)**。当语法、语义都正确后,系统就会对我们需要查询的对象加锁。这主要是为了保障数据的一致性,防止我们在查询的过程中,其他用户对这个对象的结构发生改变。
5. **数据访问权限的核对(data dict cache)**。当语法、语义通过检查之后,客户端还不一定能够取得数据。服务器进程还会检查,你所连接的用户是否有这个数据访问的权限。若连接上服务器的用户不具有数据访问权限的话,则客户端就不能够取得这些数据。
6. **确定最佳执行计划** 。当语句与语法都没有问题,权限也匹配的话,服务器进程还是不会直接对数据库文件进行查询。服务器进程会根据一定的规则,对这条语句进行优化。

### **parser.cpp解读**

**概述:  parser.cpp是PostgreSQL语法的主入口，该语法不允许执行任何表访问（因为即使在中止的事务中，我们也需要能够进行基本解析）。因此，语法返回的数据结构是“原始”解析树，仍然需要通过analyze.c和相关文件进行分析。**



raw_parser 给定一个字符串形式的查询，做词法和语法分析。返回原始（未分析）解析树的列表。

```c++
List* raw_parser(const char* str, List** query_string_locationlist)
{
    core_yyscan_t yyscanner;
    base_yy_extra_type yyextra;
    int yyresult;

    /* 重置u_sess - > parser_cxt.stmt_contains_operator_plus*/
    resetOperatorPlusFlag();

    /* 初始化伸缩扫描器 */
    yyscanner = scanner_init(str, &yyextra.core_yy_extra, ScanKeywords, NumScanKeywords);

    /* Base_yylex()只需要这么多初始化 */
    yyextra.lookahead_num = 0;

    /* 初始化bison解析器 */
    parser_init(&yyextra);

    /* 解析 */
    yyresult = base_yyparse(yyscanner);

    /* 清理(释放内存) */
    scanner_finish(yyscanner);

    if (yyresult) { /* 错误情况 */
        return NIL;
    }

    /* 通过lex获取多查询的locationlist */
    if (query_string_locationlist != NULL) {
        *query_string_locationlist = yyextra.core_yy_extra.query_string_locationlist;

        /* 处理从客户端发送的查询，最后不带分号。 */
        if (PointerIsValid(*query_string_locationlist) &&
            (size_t)lfirst_int(list_tail(*query_string_locationlist)) < (strlen(str) - 1)) {
            *query_string_locationlist = lappend_int(*query_string_locationlist, strlen(str));
        }
    }

    return yyextra.parsetree;
}
```



**对空查询的处理,主要是检查查询是否只有注释和分号**

```c++
static bool is_empty_query(char* query_string)
{
    char begin_comment[3] = "/*";
    char end_comment[3] = "*/";
    char empty_query[2] = ";";
    char* end_comment_postion = NULL;

    /* 去除字符串开头的所有空格。 */
    while (isspace((unsigned char)*query_string)) {
        query_string++;
    }

    /* 从前面去除query_string的所有注释。 */
    while (strncmp(query_string, begin_comment, 2) == 0) {
        /*
         * 由于 query_string 已经通过解析器，每当它包含开始注释时，它将包含 end_comment和end_comment_postion(在这里		     * 不能为空)
         */
        end_comment_postion = strstr(query_string, end_comment);
        query_string = end_comment_postion + 2;
        while (isspace((unsigned char)*query_string)) {
            query_string++;
        }
    }

    /* 检查query_string是否为空查询 */
    if (strcmp(query_string, empty_query) == 0) {
        return true;
    } else {
        return false;
    }
}
```

