# JavaCC


<!--more-->



> 因为最近的项目里会有大量的 SQL 生成/解析, 所以就打算自己写一个. 于是就有了今天的这篇 blog. 
> 传送门: [JavaCC](https://javacc.github.io/javacc/)

## 概述
JavaCC 的全称是 Java Compiler Compiler (没错就是俩 Compiler), 是用于 Java 应用程序的最流行的解析器生成器. 它是一个工具, 用来读取语法规范并将其转换为可以识别与语法匹配的 Java 程序. 

除了解析器生成器本身之外, JavaCC 和提供与解析器生成相关的其他标准功能, 例如树构建 (通过 JavaCC 中包含的名为 JJTree 的工具, 这个工具之后会重点用他来完成 SQL 语法解析)、操作和调试. 

## 特点
* JavaCC 的生成是自上而下 (递归下降) 解析器, 而不是像类似 YACC 的工具生成的自下而上的解析器. 这就允许使用更通用的语法, 尽管做递归是不被允许的. 自上而下的解析器还有许多的有点 (除了更通用的语法之外), 例如更容易调试, 能够解析语法中的任何非终端, 以及能够向上传递值 (属性) 并且在解析期间沿着解析树向下移动. 
* 默认情况下, JavaCC 生成一个 `LL(1)` 解析器. 但是, 可能有部分语法不是 `LL(1)`. JavaCC 提供了句法和语义前瞻的能力, 能够在这些点直接解决轮班歧义 (shift-shift ambiguities). 🌰, 解析器仅在这些点是 `LL(k)`, 但在其他的地方都保持 `LL(1)` 以获得更好的性能表现. 移位归约 (Shift-reduce) 和 reduce-reduce 冲突对于自上而下的解析器来说不是问题. 
* JavaCC 生成百分之百纯 Java 的解析器, 因此对 JavaCC 没有运行时依赖性, 也不需要在不同机器平台上运行的特殊移植工作. 
* JavaCC 允许在此法和语法规范中扩展巴克斯范式 ([extended BNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form)), 例如 `(A)*`, `(A)+` 等等. 扩展巴克斯范式在一定程度上减少了对于左递归的需求. 实际上, 扩展巴克斯范式通常更容易阅读, 例如 `A::=y(x)*` 与 `A::=Ax|y`. 
* 词法规范 (比如正则表达式、字符串) 和语法规范 (比如巴克斯范式) 都写在同一个文件中. 这样使得语法更易读, 因为可以在语法规范中内联使用正则表达式, 并且也更易于维护. 
* JavaCC 的词法分析器可以处理完整的 Unicode 输入, 并且词法规范也可以包括任何 Unicode 字符. 这个特性有助于描述语言元素, 例如允许某些 Unicode 字符 (不是 ASCII ) 但不允许其他字符的 Java 标识符. 
* JavaCC 提供了类 [Lex](https://en.wikipedia.org/wiki/Lex_(software)) 的词法状态和词法动作功能. JavaCC 中优于其他公祖的特定方面是一流状态, 它提供了诸如 `TOKEN`, `MORE`, `SKIP` 和状态更改等概念. 这就允许更清晰的规范以及来自 JavaCC 的更好的错误和警告消息. 
* 词法规范中顶一个为*特殊标记*的标记 (`Token`) 在解析期间将被忽略. 一个有用的应用是处理评论. 
	> 最后一句话, 没看明白, 原文是: **A useful application of this is in the processing of comments.** 
* 词法规范可以在整个词法规范的全局级别或在单个词法规范的基础上定义不区分大小写的标记. 
* JavaCC 带有 JJTree, 一个非常强大的树构建预处理器. 
* JavaCC 还包括 JJDoc, 这是一种将愈发文件转换为文档文件的工具, 可选择采用 HTML 格式. 
* JavaCC 提供了许多自定义它的行为和生成的解析器的行为的选项. 此类选型的示例是对输入流执行的 Unicode 处理的种类、要执行的歧义检查的标记数等. 
* JavaCC 的错误报告在解析器生成器中也是前列的. JavaCC 生成的解析器能够通过完整的诊断信息清楚地解析错误的位置. 
* 通过使用 `DEBUG_PARSER`, `DEBUG_LOOKAHEAD` 和 `DEBUG_TOKEN_MANAGER` 选项, 用户可以深入分析解析和 token 处理步骤. 
* JavaCC 版本包含范围广泛的实例, 包括 Java 和 HTML 语法. 这些实例及其文档是熟悉 JavaCC 的好方法. 

## 示例
这个示例识别匹配的大括号后跟零个或多个行终止符, 然后是文件结尾. 

* 此语法中合法字符串的示例是: 
	`{}`, `}}}` 等等
* 非法字符串的例子是:
	`{}{}`, `}{}}`, `{ }`, `{x}` 等等
	
### 语法
```java
PARSER_BEGIN(Example)
/** 简单大括号匹配器 */
public class Example {
	
	/** 主入口 */
	public static void main(String args[]) throws ParseException {
		Example parser = new Example(System.in);
		parser.Input();
	}

}

PARSER_END(Example)
	
/** 根生产 */
void Input() : 
{}
{
	"{" [ MatchedBaraces() ] "}"
}
```

### 输出
```terminal
$ java Example
<return>
```

```terminal
$ java Example
{x<return>
Lexical error at line 1, column 2.  Encountered: "x"
TokenMgrError: Lexical error at line 1, column 2.  Encountered: "x" (120), after : ""
        at ExampleTokenManager.getNextToken(ExampleTokenManager.java:146)
        at Example.getToken(Example.java:140)
        at Example.MatchedBraces(Example.java:51)
        at Example.Input(Example.java:10)
        at Example.main(Example.java:6)
```

```terminal
$ java Example
{}}<return>
ParseException: Encountered "}" at line 1, column 3.
Was expecting one of:
    <EOF>
    "\n" ...
    "\r" ...
        at Example.generateParseException(Example.java:184)
        at Example.jj_consume_token(Example.java:126)
        at Example.Input(Example.java:32)
        at Example.main(Example.java:6)
```


---

> 作者:   
> URL: https://buli-home.cn/javacc/  

