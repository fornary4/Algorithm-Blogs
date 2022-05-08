## opengaussSQL查询解析机制浅析（二）

### opengaussSQL引擎主流程图

![image-20210830232033664](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210830232033664.png)

### SQL分析器的主要工作

- 词法分析：词法分析器是一个确定有限自动机(DFA),可以按照我们定义好的词法,将输入的字符集转换为‘单词
- 语法分析：根据SQL语言的标准定义语法规则，使用词法分析中产生的词去匹配语法规则，如果一个SQL语句能够匹配一个语法规则，则生成对应的抽象语法树（Abstract Syntax Tree，AST）。
- 语义分析：对语法树（AST）进行有效性检查，检查语法树中对应的表、列、函数、表达式是否有对应的元数据，将抽象语法树转换为逻辑执行计划（关系代数表达式）。语意分析后会输出一个查询计划,这个查询计划会指导着物理执行算子一步步的运行在我们的分布式系统之上,去读取表的内容,根据SQL的语意做运算,最后输入用户的内容。语义分析阶段包含两大块,先逻辑分析后物理分析,逻辑分析基本上是纯代数的分析过程,与底层的分布式环境无关,而物理分析则是将逻辑分析后的结果做变换,与底层的执行环境密切相关。



### opengauss词法分析规则解读

在parser模块中，scan.l文件文件定义了opengauss数据库的词法规则，作用是词法分析，这个文件中的规则与psql的lexer保持同步。规则设计的一大特点是扫描器永远不会进行回溯，从某种意义上说，总有一个规则可以匹配到目前为止消耗的输入。

**词法分析错误报告**

消息的光标位置是 YYLLOC 上次设置的任何值，即，如果在 yylex() 中调用，则为当前标记的开始，如果从语法中调用，则为最近词法分析的标记。这对于来自 Bison 解析器的语法错误消息是可以的，因为一旦到达第一个无法解析的标记，Bison 解析器就会报告错误。不要将 yyerror 用于其他目的，因为光标位置可能会产生误导。

```c++
void scanner_yyerror(const char *message, core_yyscan_t yyscanner)
{
	const char *loc = yyextra->scanbuf + *yylloc;

	if (*loc == YY_END_OF_BUFFER_CHAR)
	{
		ereport(ERROR,
				(errcode(ERRCODE_SYNTAX_ERROR),
				 /* %s通常是“语法错误”的翻译 */
				 errmsg("%s at end of input", _(message)),
				 lexer_errposition()));
	}
	else
	{
		ereport(ERROR,
				(errcode(ERRCODE_SYNTAX_ERROR),
				 /* translator: 第一个%s通常是语法错误的翻译 */
				 errmsg("%s at or near \"%s\"", _(message), loc),
				 lexer_errposition()));
	}
}
```

**对分隔符的处理**

为了使 Windows 和 Mac 客户端以及 Unix 客户端的输入安全，opengauss接受 \n 或 \r 作为换行符。 DOS 样式的 \r\n 序列将被视为两个连续的换行符，但这不会引起任何问题。以 -- 开头并扩展到下一个换行符的注释被视为等效于单个空白字符。

```c++
/* 分隔符的定义 */
space			[ \t\n\r\f]
horiz_space		[ \t\f]
newline			[\n\r]
non_newline		[^\n\r]

comment			("--"{non_newline}*)

whitespace		({space}+|{comment})
    
special_whitespace		({space}+|{comment}{newline})
horiz_whitespace		({horiz_space}|{comment})
whitespace_with_newline	({horiz_whitespace}*{newline}{special_whitespace}*)
```



**词法分析完成后调用scanner_finish，以便在scananner_init()之后进行清理**

```c++
void scanner_finish(core_yyscan_t yyscanner)
{
	if (t_thrd.postgres_cxt.clear_key_memory)
	{
		errno_t rc = EOK;
		memset(yyextra->scanbuf, 0x7F, yyextra->scanbuflen);
		*(volatile char*)(yyextra->scanbuf) = *(volatile char*)(yyextra->scanbuf);
		rc = memset_s(yyextra->literalbuf, yyextra->literallen, 0x7F, yyextra->literallen);
		securec_check(rc, "\0", "\0");
	}

	/*
	 * 我们不需要调用 yylex_destroy()，因为它所做的只是释放少量的控制存储。在解析上下文被破坏之前泄漏存储更便宜。无论如何，与	   * 输出解析树相比，所涉及的空间量通常可以忽略不计。
	 */
	if (yyextra->scanbuflen >= 8192)
		FREE_POINTER(yyextra->scanbuf);
	if (yyextra->literalalloc >= 8192)
		FREE_POINTER(yyextra->literalbuf);
	if (yyextra->parameter_list)
	{
		list_free_deep(yyextra->parameter_list);
		yyextra->parameter_list = NIL;
	}
}

```

