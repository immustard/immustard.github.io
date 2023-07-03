# JJTree


<!--more-->

> 传送门: [JJTree](https://javacc.github.io/javacc/documentation/jjtree.html)

## 概述
JJTree 是 JavaCC 的预处理器, 可在 JavaCC 源代码的不同位置插入解析树构建操作. JJTree 的输出通过 JavaCC 运行以创建解析器. 此文档描述了如何使用 JJTree, 以及如何将解析器连接到 JJTree. 

默认情况下, JJTree 生成代码来为语言中的每个非终节点结构解析书节点. 但是可以修改这个行为, 为了某些非终节点不会生成节点, 或者为生产扩展的一部分生成节点. 

JJTree 定义了一个 Java 接口 `Node` , 并且所有解析树节点都必须实现它. 这个接口提供了例如设置父节点、添加和检索子节点等操作的方法. 

JJTree 以两种模式之一运行, 简单模式和多模式 (需要更好的术语). 在简单模式下, 每个解析树节点都是具体类型的 `SimpleNode`, 在多模式下, 解析树节点的类型源自节点的名称. 如果不提供节点类的实现, 那么 JJTree 会生成基于 `SimpleNode` 的示例实现. 之后可以修改该实现来进行适配. 

虽然 JavaCC 是一个自上而下的解析器, 但是 JJTree 是自下而上地构造解析树. 为此, 它使用一个堆栈, 在创建节点后推送节点. 当它为它们找到父级时, 它会从堆栈中弹出 (pop) 子级并将它们添加到父级, 并且最后推送新的父节点本身. 堆栈是开放的, 这意味着可以从语法操作中访问它: 可以用认为合适的方式进行推送 (push), 弹出 (pop)和其他方式操作其内容 (详细信息可以参考 [节点范围和用户操作](#节点范围和用户操作) )

JJTree 提供了两种基本类型的节点的装饰, 以及一些语法速记以方便它们的使用. 

#### 确定节点
一个确定节点是用特定数量的子节点构成的. 许多节点从堆栈中弹出并成为新节点的子节点, 然后将其推送到堆栈本身上. 

可以标记一个确定节点, 类似这样: 
```
#ADefiniteNode(INTEGER EXPRESSION)
```

确定节点描述符表达式可以是任何整数表达式, 尽管文字整数常量是迄今为止最常见的表达式. 

#### 条件节点
当且仅当条件计算为真 (`true`) 时才使用其节点范围内被推送到堆栈的所有子节点构造条件节点. 如果计算条件为假 (`false`) 时, 该节点不会被构造, 并且其所有子节点会留在节点堆栈中. 

可以标记一个条件节点, 类似这样: 
```
#ConditionalNode(BOOLEAN EXPRESSION)
```

条件节点描述符表达式可以是任何布尔表达式. 

 条件节点有两种常见的简写:
 	1. 不定节点 (Indefinite nodes)
		```
		#IndefiniteNode is short for #IndefiniteNode(true)
		```
	2. 大于节点 (Greater-than nodes)
		```
		#GTNode(>1) is short for #GTNode(jjtree.arity() > 1)
		```
	
不定节点速记 `(1)` 在后面跟着带括号的展开时可能会导致 JJTree 源中的歧义. 在这种情况下, 速记必须替换为完整表达式:
```
( ... ) #N ( a() )
```
上面这个是有歧义的, 所以必须使用显示条件:
```
( ... ) #N(true) ( a() )
```

**注意: 节点描述符表达式不应该有副作用. JJTree没有指定表达式将被计算多少次.**

默认情况下, JJTree 将每个非终节点视为不定节点, 并从其生产名称派生节点的名称. 可以使用以下语法为其命名: 
```
void P1() #MyNode : { ... } { ... }
```

当解析器识别出一个 `P1` 非终节点时, 它会开始一个不定节点. 它标记堆栈, 以便在 `P1` 扩展中由非终节点创建并推送到堆栈上的任何解析树节点都将被弹出并成为节点 `MyNode` 的子节点. 

如果要禁止为生产创建节点, 可以使用以下语法: 
```
void P2() #void : { ... } { ... }
```

现在, 在 `P2` 的扩展中由非终节点推送的任何解析树节点都将保留在堆栈上, 以便弹出并使其成为树上更向上的生产子节点. 可以使用 `NODE_DEFAULT_VOID` 选项将此设置为未装饰节点的默认行为. 

```
void P3() : {} {
	P4() ( P5() )+ P6()
}
```

在这个例子中, 开始一个不定节点 `P3`, 标记堆栈, 然后解析一个 `P4` 节点、一个或多个 `P5` 节点和一个 `P6` 节点. 

可以进一步自定义生成的树: 
```
void P3() : {} {
	P4() ( P5() )+ #ListOfP5s P6()
}
```

现在, `P3` 节点将有一个 `P4` 节点、一个 `ListOfP5s` 节点和一个 `P6` 节点作为子节点. `#Name` 构造充当后缀运算符, 其作用域是紧接在前的扩展单元. 

### 节点范围和用户操作
每个节点都与一个节点作用域 (*node scope*) 相关联. 此范围内的用户操作可以通过使用特殊标识符 `jjtThis` 来访问正在构建的节点, 以引用该节点. 该标识符被隐式声明为该节点的正确类型, 因此可以轻松地访问该节点拥有的任何字段和方法. 

作用域是紧靠在节点修饰之前的扩展单元. 这可以是带括号的表达式. 当对生产签名进行装饰时 (可能隐式带有默认节点), 作用域是生产的整个右侧, 包括其声明块. 

还可以在扩展引用的左侧使用涉及 `jjtThis` 的表达式. 

🌰:
```
... ( jjtThis.my_foo = foo() ) #Baz ...
```

这里的 `jjtThis` 指向的是一个 `Baz` 节点, 他有一个名为 `my_foo` 的字段. 将解析生成的 `foo()` 的结果赋给 `myfoo`. 

节点范围内的最终用户操作与所有其他操作不同. 当其中的代码执行时, 节点的子节点已经从堆栈中弹出并添加到节点中, 节点本身已被推送到堆栈中. 现在, 可以通过节点的方法 (比如 `jjtGetChild()`) 来访问子节点. 

除最后一个操作之外的其他用户操作只能访问堆栈上的子级. 它们还没有被加入到节点中, 所以还不能通过节点的方法进行访问. 如果条件节点的节点描述符表达式的计算结果为 `false`, 则不会将其添加到堆栈中, 也不会向其添加子节点. 条件节点范围内的最终用户操作可以通过调用 `nodeCreated()` 方法来确定节点是否被创建. 只有当节点的条件被满足且被创建且被推入到节点堆栈中时才返回 `true`, 否则返回 `false`. 

### 异常处理
由节点作用域中的扩展抛出的异常不在该节点作用域内捕获, 由 JJTree 本身捕获. 发生这种情况时, 任何已被压入节点范围内的堆栈的节点都将被弹出并丢弃. 然后重新抛出异常. 

这么做的目的是使解析器能够实现错误恢复, 并在已知状态下继续使用节点堆栈. 

**注意: JJTree 目前无法检测节点作用域内的用户操作是否引发异常. 这样的异常可能会被错误地处理.**

### 节点作用域钩子
如果 `NODE_SCOPE_HOOK` 设置为 `true`, 那么 JJTree 将在每个节点作用域的入口和出口上生成对两个用户定义的解析器方法的调用. 这些方法必须具有以下声明:
```java
void jjtreeOpenNodeScope(Node n)
void jjtreeCloseNodeScope(Node n)
```

如果解析器是 `STATIC`, 那么这些方法也必须被声明为 `static`. 它们都以当前节点作为参数调用. 

一种用途可能是将解析器对象本身存储在节点中, 以便可以提供应该由该解析器生成的所有节点共享的状态. 🌰, 解析器可能维护一个符号表. 

```java
void jjtreeOpenNodeScope(Node n) {
	((SimpleNode)n).jjtSetValue(getSymbolTable());
}

void jjtreeCloseNodeScope(Node n) {}
```

其中 `getSymbolTable()` 是一个用户自定义的方法, 用来返回节点的符号表的结构. 

### 跟踪令牌
跟踪每个节点的最初和最后的令牌通常很有用, 这样就可以轻松地再次复制输入. 通过设置 `TRACK_TOKENS` 选项, 生成的 `SimpleNode` 类将包含 4 个额外的方法:
```java
public Token jjtGetFirstToken()
public void jjtSetFirstToken(Token token)
public Token jjtGetLastToken()
public void jjtSetLastToken(Token token)
```

当解析器运行的时候, 将自动设置每个节点的最初和最后的令牌. 

### 节点的生命周期
节点在创建的时候将进行一系列的确定的步骤. 这是从节点本身的角度来看的序列:

1. 使用唯一的整数参数调用节点的构造函数. 此参数标识节点的类型, 在简单模式下特别有用. JJTree 会自动生成一个叫做 `<parser>TreeConstants.java` 的文件, 用来声明有效常量. 常量的命名是通过在节点的大写名称前加上 JJT 来派生的, 点 (`.`) 替换为下划线 (`_`). 为方便起见, 在同一文件中维护了一个名为 `jjtNodeName[]` 的字符串数组, 它将常量映射到未修改的节点名称. 
2. 节点的 `jjtOpen()` 方法被调用. 
3. 如果设置了 `NODE_SCOPE_HOOK` 选项, 用户自定义解析器方法 `openNodeScope()` 会被调用, 并且节点作为参数进行传递. 该方法可以初始化节点中的字段或调用其方法. 🌰, 它可以存储节点的第一个令牌. 
4. 如果在解析节点时抛出未处理的异常, 则将该节点抛弃. JJTree 再也不会引用它了. 它不会被关闭, 并且**不会使用**它作为参数来调用用户定义的节点作用域钩子 `CloseNodeHook()`. 
5. 否则, 如果该节点是有条件的, 并且其条件表达式的计算结果为 `false`, 则该节点将被放弃. 它不会被关闭, 尽管**可以使用**它作为参数调用用户定义的节点作用域钩子 `CloseNodeHook()`. 
6. 否则, 由确定节点的整数表达式指定的节点的所有子节点, 或在条件节点范围内推送到堆栈上的所有节点都将添加到该节点. 它们添加的顺序是不确定的. 
7. 节点的 `jjtClose()` 方法被调用. 
8. 节点被推到堆栈中. 
9. 如果设置了 `NODE_SCOPE_HOOK` 选项, 用户自定义解析器方法 `closeNodeScope()` 会被调用, 并且节点作为参数进行传递.
10. 如果节点不是根节点, 那么它将被添加为另一个节点的子节点, 并调用其 `jjtSetParent()` 方法. 

### 访客支持
JJTree 为访问者设计模式提供了一些基本支持. 如果 `VISITOR` 选项设置为 `true`, JJTree 将把 `jjtAccept()` 方法插入到它生成的所有节点类中, 并且还生成可以实现并传递给节点以接受的访问者接口. 

访问者接口的命名是通过将 `Visitor` 附加到解析器的名称来构造的. 每次运行 JJTree 时都会重新生成接口, 以便准确表示解析器使用的节点集. 如果没有为新节点更新实现类, 这将导致编译时错误. 这是一个功能. 

### 选项
| **选项**                  | **缺省值**           | **描述**                                                     |
| ------------------------- | -------------------- | ------------------------------------------------------------ |
| `BUILD_NODE_FILES`        | `true`               | 为 `SimpleNode` 和语法中使用的任何其他节点生成示例实现.      |
| `MULTI`                   | `false`              | 生成多模式解析树. 缺省值为 `false`, 生成一个简单模式解析树.  |
| `NODE_DEFAULT_VOID`       | `false`              | 与其使每个未装饰的产品成为不定节点, 不如使其无效.            |
| `NODE_CLASS`              | `""`                 | 如果设置定义将扩展 `SimpleNode` 的用户提供的类名称, 那么创建的任何树节点都将是 `NODE_CLASS` 的子类. |
| `NODE_FACTORY`            | `""`                 | 指定一个包含工厂方法的类, 该方法具有以下签名以构造节点: `public static Node jjtCreate(int id)`. 为了向后兼容, 也可以指定值 `false`, 这意味着 `SimpleNode` 将用作工厂类. |
| `NODE_PACKAGE`            | `""`                 | 生成节点类的包. 默认为解析器包.                              |
| `NODE_EXTENDS`            | `""`                 | 弃用. `SimpleNode` 类的超类. 通过提供自定义超类, 可以避免编辑生成的`SimpleNode.java`. |
| `NODE_PREFIX`             | `"AST"`              | 用于在多模式下从节点标识符构造节点类名的前缀.                |
| `NODE_SCOPE_HOOK`         | `false`              | 在每个节点作用域的入口和出口上生成对两个用户定义的解析器方法的调用. |
| `NODE_USES_PARSER`        | `false`              | JJTree 将使用另一种形式的节点构造例程, 在其中传递解析器对象. 🌰: <br/>`public static Node MyNode.jjtCreate(MyParser p, int id);`<br/>`MyNode(MyParser p, int id);` |
| `TRACK_TOKENS`            | `false`              | 在 `SimpleNode` 中插入 `jjtGetFirstToken()`, `jjtSetFirstToken()`, `getLastToken()` 和 `jjtSetLastToken()` 方法. `FirstToken` 在进入节点作用域时自动设置; `LastToken` 在退出节点作用域时自动设置. |
| `STATIC`                  | `true`               | 为静态解析器生成代码. 这必须与等效的 JavaCC 选项一致使用. 该选项的值在 JavaCC 源中发出. |
| `VISITOR`                 | `false`              | 在节点类中插入一个 `jjtAccept()` 方法, 并为语法中使用的每个节点类型生成一个包含条目的访问者实现. |
| `VISITOR_DATA_TYPE`       | `"Object"`           | 如果设置了该选项, 它将在生成的`jjtAccept()` 方法和 `visit()` 方法的签名中用作数据参数的类型. |
| `VISITOR_RETURN_TYPE`     | `"Object"`           | 如果设置了该选项, 它将在生成的`jjtAccept()` 方法和 `visit()` 方法的签名中用作返回参数的类型. |
| `VISITOR_EXCEPTION`       | `""`                 | 如果设置了该选项, 它将在生成的`jjtAccept()` 方法和 `visit()` 方法的签名中. |
| `JJTREE_OUTPUT_DIRECTORY` | `"OUTPUT_DIRECTORY"` | 默认情况下, JJTree 在全局 `OUTPUT_DIRECTORY` 设置中指定的目录中生成其输出. 显式设置此选项允许用户将解析器与树文件分开. |

## JJTree 接口

### JJTree 状态
JJTree 将其状态保存在一个名为 `jjtree` 的解析器类字段中. 可以使用此成员中的方法来操作节点堆栈. 

```java
final class JJTreeState {
	// 调用它来重新初始化节点堆栈
	void reset();
	
	// 返回抽象语法树 (AST) 的根节点
	Node rootNode();
	
	// 判断当前节点是否实际关闭并推送
	boolean nodeCreated();
	
	// 返回节点上当前推送的节点数
	// 当前节点作用域内的堆栈
	int arity();
	
	// 推送节点到堆栈
	void pushNode(Node n);
	
	// 返回堆栈顶部的节点, 并将其从堆栈中移除
	Node popNode();
	
	// 返回当前位于堆栈顶部的节点
	Node peekNode();
}
```

### 节点对象
```java
/**
	* 所有 AST 节点必须实现该接口.
	* 它为构建节点之间的父子关系提供了基本机制.
	*/
public interface Node {
	// 将节点设为当前节点后调用此方法.
	// 表示现在可以向其中添加子节点. 
	public void jjtOpen();
	
	// 添加所有子节点后调用此方法.
	public void jjtClose();
	
	// 这对方法用于通知节点其父节点.
	public void jjtSetParent(Node n);
	public Node jjtGetParent();
	
	// 此方法通知节点将其参数添加到节点的子节点列表中. 
	public void jjtAddChild(Node n, int i);
	
	// 此方法返回子节点.
	// 子节点编号从 0 开始, 从左至右.
	public Node jjtGetChild(int i);
	
	// 返回节点拥有的子节点数.
	int jjtGetNumChildren();
}
```

`SimpleNode` 类实现了 `Node` 接口, 如果它不存在, 则由 JJTree 自动生成. 可以使用该类作为节点实现的模板或超类, 或修改其进行适配. `SimpleNode` 还提供了递归转储节点及其子节点的基本机制. 

可以像这样使用这个命令: 
```java
{
	((SimpleNode)jjtree.rootNode()).dump(">");
}
```

`dump()` 的字符串参数用作填充, 以指示树层次结构. 

如果设置了 `VISITOR` 选项, 则会生成另一个实用程序方法: 
```java
{
	public void childrenAccept(MyParserVisitor visitor);
}
```

这会轮流遍历节点的子节点, 要求它们接受访问者. 这在实现前序和后序遍历时很有用. 


---

> 作者:   
> URL: https://buli-home.cn/jjtree/  

