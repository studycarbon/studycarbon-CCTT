# 语言设计

> **地位：**截至2022年8月9日的最新消息，包括[截至#1327](https://github.com/carbon-language/carbon-lang/pull/1327)的提案。

## 目录

- 介绍
  - [本文档是临时的](#this-document-is-provisional)
  - [基础知识之旅](#tour-of-the-basics)
- [代码和注释](#code-and-comments)
- [构建模式](#build-modes)
- [类型是值](#types-are-values)
- 基元类型
  - [`bool`](#bool)
  - 整数类型
    - [整数文本](#integer-literals)
  - 浮点类型
    - [浮点文本](#floating-point-literals)
  - 字符串类型
    - [字符串](#string-literals)
- [价值类别和价值阶段](#value-categories-and-value-phases)
- 复合类型
  - [元组](#tuples)
  - [结构类型](#struct-types)
  - [指针类型](#pointer-types)
  - [数组和切片](#arrays-and-slices)
- [表达 式](#expressions)
- [声明、定义和范围](#declarations-definitions-and-scopes)
- 模式
  - [绑定模式](#binding-patterns)
  - [解构模式](#destructuring-patterns)
  - [可反驳的模式](#refutable-patterns)
- 名称绑定声明
  - [常量 `let` 声明](#constant-let-declarations)
  - [变量`变量`声明](#variable-var-declarations)
  - [`auto`](#auto)
- 功能
  - [参数](#parameters)
  - [`自动`返回类型](#auto-return-type)
  - [块和语句](#blocks-and-statements)
  - [转让声明](#assignment-statements)
  - 控制流
    - [`如果` 和 `否则`](#if-and-else)
    - 循环
      - [`while`](#while)
      - [`for`](#for)
      - [`break`](#break)
      - [`continue`](#continue)
    - `return`
      - [`returned var`](#returned-var)
    - [`match`](#match)
- 用户定义类型
  - 类
    - [分配](#assignment)
    - [类函数和工厂函数](#class-functions-and-factory-functions)
    - [方法](#methods)
    - [遗产](#inheritance)
    - [存取控制](#access-control)
    - [析 构 函数](#destructors)
    - [`const`](#const)
    - [未形成状态](#unformed-state)
    - [移动](#move)
    - [米辛斯](#mixins)
  - [选择类型](#choice-types)
- 名字
  - [文件、库、包](#files-libraries-packages)
  - [包装声明](#package-declaration)
  - [进口](#imports)
  - [名称可见性](#name-visibility)
  - [包范围](#package-scope)
  - [命名空间](#namespaces)
  - [命名约定](#naming-conventions)
  - [别名](#aliases)
  - 名称查找
    - [常见类型的名称查找](#name-lookup-for-common-types)
- 泛 型
  - [已检查的参数和模板参数](#checked-and-template-parameters)
  - [接口和实现](#interfaces-and-implementations)
  - [组合约束](#combining-constraints)
  - [关联类型](#associated-types)
  - 通用实体
    - [泛型类](#generic-classes)
    - [泛型选择类型](#generic-choice-types)
    - [通用接口](#generic-interfaces)
    - [泛型实现](#generic-implementations)
  - [其他特点](#other-features)
  - [泛型类型相等和`观察`声明](#generic-type-equality-and-observe-declarations)
  - 运算符过载
    - [通用类型](#common-type)
- 与 C 和 C++ 的双向互操作性
  - [目标](#goals)
  - [非目标](#non-goals)
  - [导入和`#include`](#importing-and-include)
  - [ABI 和动态链接](#abi-and-dynamic-linking)
  - [运算符过载](#operator-overloading-1)
  - [模板](#templates)
  - [标准类型](#standard-types)
  - [遗产](#inheritance-1)
  - [枚举](#enums)
- 未完成的故事
  - [安全](#safety)
  - [生存期和移动语义](#lifetime-and-move-semantics)
  - [元编程](#metaprogramming)
  - [模式匹配作为函数重载分辨率](#pattern-matching-as-function-overload-resolution)
  - [错误处理](#error-handling)
  - 执行抽象
    - [抽象机器和执行模型](#abstract-machine-and-execution-model)
    - [Lambdas](#lambdas)
    - [共同例程](#co-routines)
    - [并发](#concurrency)

## 介绍

本文档描述了Carbon语言的设计，以及该设计的基本原理。本文档概述了Carbon项目的当前状态，是为Carbon的构建者和那些有兴趣了解有关Carbon的更多信息的人编写的。

本文档*不是*完整的编程手册，也不为设计决策提供详细而全面的理由。这些描述可以在链接的专用设计中找到。

### 本文档是临时的

本文档包含许多临时或占位符。这意味着所使用的语法，语言规则，标准库以及设计的其他方面都有尚未通过Carbon过程决定的内容。这种初步材料填补了空白，直到可以填补设计的各个方面。临时功能已尽最大努力标记为此类功能。

### 基础知识之旅

下面是一个简单的函数，显示了一些碳代码：

```
import Math;

// Returns the smallest factor of `n` > 1, and
// whether `n` itself is prime.
fn SmallestFactor(n: i32) -> (i32, bool) {
  let limit: i32 = Math.Sqrt(n) as i32;
  var i: i32 = 2;
  while (i <= limit) {
    let remainder: i32 = n % i;
    if (remainder == 0) {
      Carbon.Print("{0} is a factor of {1}", i, n);
      return (i, false);
    }
    if (i == 2) {
      i = 3;
    } else {
      // Skip even numbers once we get past `2`.
      i += 2;
    }
  }
  return (n, true);
}
```

Carbon是一种C++和C开发人员应该熟悉的语言。此示例具有熟悉的构造，如[导入](#imports)、[注释](#code-and-comments)、[函数定义](#functions)、[类型化参数](#binding-patterns)和[表达式](#expressions)。[语句](#blocks-and-statements)和[声明](#declarations-definitions-and-scopes)以大括号中的 a 或某些内容结尾....`;``{``}`

与C或C++不同的其他一些功能可能会脱颖而出。首先，[声明以](#declarations-definitions-and-scopes)介绍人关键字开头。 引入了一个函数声明，并引入了一个[变量声明](#variable-var-declarations)。`fn``var`

该示例从[`导入`声明](#imports)开始。碳导入更像是[C++模块](https://en.cppreference.com/w/cpp/language/modules)[，而不是使用`#include`进行预处理时的文本包含](https://en.cppreference.com/w/cpp/preprocessor/include)。声明[从包中导入库](#files-libraries-packages)。它必须出现在碳源文件的顶部，这是[可选`包装`声明](#package-declaration)之后的第一件事。可以选择将库拆分为 [api 和实现文件](#files-libraries-packages)，如C++头文件和源文件，但在任何情况下都不需要源文件。以下声明来自示例：`import`

```
import Math;
```

从包 中导入默认库。此库中的名称可作为 的成员进行访问，如 。该函数来自包的前奏库，该库是[默认导入的](#name-lookup-for-common-types)。与C++不同，不同包的命名空间保持独立，因此没有名称冲突。`Math``Math``Math.Sqrt``Carbon.Print``Carbon`

碳[注释](#code-and-comments)必须单独放在一行上，以以下开头：`//`

```
// Returns the smallest factor of `n` > 1, and
// whether `n` itself is prime.
...
      // Skip even numbers once we get past `2`.
```

[函数定义](#functions)包括：

- 关键字介绍人，`fn`
- 函数的名称，
- 圆形帕伦斯的参数列表 ...，`(``)`
- 可选和返回类型，以及`->`
- 卷曲大括号内的身体....`{``}`

```
fn SmallestFactor(n: i32) -> (i32, bool) {
  ...
      return (i, false);
  ...
  return (n, true);
}
```

函数的主体是[语句](#blocks-and-statements)和[声明](#declarations-definitions-and-scopes)的有序序列。当函数执行到达语句或函数体的末尾时结束。 语句还可以指定返回其值的表达式。`return``return`

此处引用有符号[整数类型](#integer-types)，具有 32 位，并且是[布尔类型](#bool)。碳也有[浮点类型](#floating-point-types)，如 和 和 和 [字符串类型](#string-types)。`i32``bool``f32``f64`

[变量声明](#variable-var-declarations)由三部分组成：

- 关键字介绍人，`var`
- 名称后跟 a 和类型，其声明方式与函数签名中的参数相同，以及`:`
- 可选的初始值设定项。

```
  var i: i32 = 2;
```

您可以使用[赋值语句](#assignment-statements)修改变量的值：

```
      i = 3;
      ...
      i += 2;
```

[常量是](#constant-let-declarations)使用关键字引入符声明的。语法与变量声明平行，只是初始值设定项是必需的：`let`

```
  let limit: i32 = Math.Sqrt(n) as i32;
  ...
    let remainder: i32 = n % i;
```

初始值设定项是一个[表达式](#expressions)。它首先调用函数作为参数。然后，运算符将浮点返回值强制转换为 。像这样的有损转换必须明确完成。`Math.Sqrt(n) as i32``Math.Sqrt``n``as``i32`

其他表达式包括 ，它应用带有 和 作为参数的二元模运算符，以及 应用比较运算符产生结果。当表达式用作语句时，将忽略表达式返回值，如以下对函数的调用所示：`n % i``%``n``i``remainder == 0``==``bool``Carbon.Print`

```
      Carbon.Print("{0} is a factor of {1}", i, n);
```

函数调用由函数名称后跟圆括号中的逗号分隔参数列表组成。...`(``)`

控制流语句（包括 、、、和 ）更改语句的执行顺序，就像它们在C++中所做的那样：`if``while``for``break``continue`

```
  while (i <= limit) {
    ...
    if (remainder == 0) {
      ...
    }
    if (i == 2) {
      ...
    } else {
      ...
    }
  }
```

大括号中的每个代码块...定义一个范围。名称从其声明中可见，直到包含它的最内层作用域的末尾。因此，在示例中，直到大括号关闭 .`{``}``remainder``}``while`

示例函数使用[*元组*](#tuples)（[复合类型）](#composite-types)返回多个值。元组值和类型都是使用括号内的逗号分隔列表编写的。所以 和 是元组值，并且是它们的类型。`(i, false)``(n, true)``(i32, bool)`

[结构类型](#struct-types)类似，只是它们的成员是按名称而不是位置引用的。该示例可以更改为使用结构，如下所示：

```
// Return type of `{.factor: i32, .prime: bool}` is a struct
// with an `i32` field named `.factor`, and a `bool` field
// named `.prime`.
fn SmallestFactor(n: i32) -> {.factor: i32, .prime: bool} {
  ...
    if (remainder == 0) {
      // Return a struct value.
      return {.factor = i, .prime = false};
    }
  ...
  // Return a struct value.
  return {.factor = n, .prime = true};
}
```

## 代码和注释

所有源代码均为 UTF-8 编码文本。注释、标识符和字符串允许包含非 ASCII 字符。

```
var résultat: String = "Succès";
```

注释以两个斜杠开头，然后转到行尾。它们必须是行上唯一的非空格。`//`

```
// Compute an approximation of π
```

> 引用：
>
> - [源文件](code_and_name_organization/source_files.md)
> - [词汇约定](lexical_conventions)
> - 提案 [#142：Unicode 源文件](https://github.com/carbon-language/carbon-lang/pull/142)
> - 提案[#198：评论](https://github.com/carbon-language/carbon-lang/pull/198)

## 构建模式

Carbon 编译器的行为取决于*构建模式*：

- 在*开发版本中*，优先级是诊断问题和缩短构建时间。
- 在*性能版本中*，优先级是最快的执行时间和最低的内存使用率。
- 在*强化构建*中，首要任务是安全性，其次是性能。

> 参考：[安全策略]()

## 类型是值

表达式在 Carbon 中计算值，这些值始终是强类型化的，就像在C++中一样。但是，与C++的一个重要区别是，类型本身被建模为值。具体来说，就是编译时常量值。这会产生许多后果：

- 类型名称位于与函数、变量、命名空间等共享的同一命名空间中。
- 用于编写类型的语法是[表达式](#expressions)语法，而不是类型的单独语法。因此，Carbon不使用尖括号...在类型中，因为 和 用于表达式中的比较。`<``>``<``>`
- 函数调用语法用于指定类型的参数，如 。`HashMap(String, i64)`

某些值（如 and ）甚至可以用作类型，但仅在处于类型位置时才像类型一样运行，例如在变量声明中的 a 之后或在函数声明中的 返回类型之后。类型位置中的任何表达式都必须[是常量或符号值](#value-categories-and-value-phases)，以便编译器可以确定该值是否可以用作类型。这也限制了操作员可以对类型执行不同操作的程度。这有利于一致性，但对Carbon的设计是一个重大限制。`()``{}``:``->`

## 基元类型

基元类型分为以下几类：

- 布尔类型 ，`bool`
- 有符号和无符号整数类型，
- IEEE-754 浮点类型，以及
- 字符串类型。

这些都是通过[前奏](#name-lookup-for-common-types)提供的。

> 引用：[基元类型](primitive_types.md)

### `bool`

该类型是具有两个可能值的布尔类型：和 。[比较表达式生成](#expressions)值。[控制流语句](#control-flow)中的条件参数（如 [`if`](#if-and-else) 和 [`while`](#while)）以及 [`if-then-else` 条件表达式````](#expressions)采用值。`bool``true``false``bool``bool`

### 整数类型

具有位宽度的有符号整数类型可以写成 。为方便和简洁起见，常见的二次幂大小可以写上后跟大小：、、或 。有符号整数[溢出](expressions/arithmetic.md#overflow-and-other-error-conditions)是一个编程错误：`N``Carbon.Int(N)``i``i8``i16``i32``i64``i128`

- 在开发版本中，溢出在运行时发生时将立即捕获。
- 在性能生成中，优化程序可以假定不会发生此类情况。因此，如果他们这样做，则不会定义程序的行为。
- 在强化的版本中，溢出不会导致未定义的行为。相反，要么程序中止，要么算术将计算为数学上不正确的结果，例如二的补码结果或零。

无符号整数类型包括：、、、、、和 。无符号整数类型在溢出时换行，我们强烈建议不要使用它们，除非需要这些语义。这些类型用于位操作或模块化算术，常见于[哈希](https://en.wikipedia.org/wiki/Hash_function)、[加密](https://en.wikipedia.org/wiki/Cryptography)和 [PRNG](https://en.wikipedia.org/wiki/Pseudorandom_number_generator) 用例中。永远不能为负的值（如大小），但包装没有意义[的值应使用有符号整数类型]()。`u8``u16``u32``u64``u128``Carbon.UInt(N)`

> 引用：
>
> - 针对潜在客户的问题 [#543：为固定大小的整数类型选择名称](https://github.com/carbon-language/carbon-lang/issues/543)
> - 提案 [#820：隐式转换](https://github.com/carbon-language/carbon-lang/pull/820)
> - 提案 [#1083：算术表达式](https://github.com/carbon-language/carbon-lang/pull/1083)

#### 整数文本

整数可以写成十进制、十六进制或二进制：

- `12345`（十进制）
- `0x1FE`（十六进制）
- `0b1010`（二进制）

下划线 （） 可用作数字分隔符。数字文本区分大小写：，必须为小写，而十六进制数字必须为大写。整数文本从不包含 .`_``0x``0b``.`

与C++不同，文本没有表示其类型的后缀。相反，数字文本具有从其值派生的类型，并且可以[隐式转换为](expressions/implicit_conversions.md)可以表示该值的任何类型。

> 引用：
>
> - [整数文本](lexical_conventions/numeric_literals.md#integer-literals)
> - 提案[#143：数字文字](https://github.com/carbon-language/carbon-lang/pull/143)
> - 提案[#144：数字文字语义](https://github.com/carbon-language/carbon-lang/pull/144)
> - 提案 [#820：隐式转换](https://github.com/carbon-language/carbon-lang/pull/820)
> - 提案 [#1983：削弱数字分隔符放置规则](https://github.com/carbon-language/carbon-lang/pull/1983)

### 浮点类型

Carbon 中的浮点类型具有 IEEE 754 语义，使用舍入到最近舍入模式，并且不设置任何浮点异常状态。它们以 和 位数命名：、、、和 。还提供了[`BFloat16`](primitive_types.md#bfloat16)。`f``f16``f32``f64``f128`

> 引用：
>
> - 针对潜在客户的问题 [#543：为固定大小的整数类型选择名称](https://github.com/carbon-language/carbon-lang/issues/543)
> - 提案 [#820：隐式转换](https://github.com/carbon-language/carbon-lang/pull/820)
> - 提案 [#1083：算术表达式](https://github.com/carbon-language/carbon-lang/pull/1083)

#### 浮点文本

浮点类型以及[用户定义类型](#user-defined-types)可以从*实数文本*初始化。支持十进制和十六进制实数文本：

- `123.456`（数字在 两侧`.`)
- `123.456e789`（可选或之后`+``-``e`)
- `0x1.Ap123`（可选或之后`+``-``p`)

与整数文本一样，下划线 （） 可用作数字分隔符。实数文本始终有一个句点 （） 和一个数字在句点的每一侧。当实数文本被解释为浮点类型的值时，其值是最接近文本值的可表示实数。在平局的情况下，选择尾数为偶数的最接近的值。`_``.`

> 引用：
>
> - [实数文本](lexical_conventions/numeric_literals.md#real-number-literals)
> - 提案[#143：数字文字](https://github.com/carbon-language/carbon-lang/pull/143)
> - 提案[#144：数字文字语义](https://github.com/carbon-language/carbon-lang/pull/144)
> - 提案 [#820：隐式转换](https://github.com/carbon-language/carbon-lang/pull/820)
> - 提案 [#866：允许在浮动文本中建立联系](https://github.com/carbon-language/carbon-lang/pull/866)
> - 提案 [#1983：削弱数字分隔符放置规则](https://github.com/carbon-language/carbon-lang/pull/1983)

### 字符串类型

> **注意：**这是临时性的，字符串类型的设计尚未通过提案流程。

有两种字符串类型：

- `String`- 被视为包含 UTF-8 编码文本的字节序列。
- `StringView`- 对被视为包含 UTF-8 编码文本的字节序列的只读引用。

#### 字符串

字符串文本可以在字符串的开头和结尾使用双引号 （） 写在单行上，如 中所示。`"``"example"`

多行字符串文本（称为*块字符串文本）*以三个双引号 （） 开头和结尾，并且可能在第一个 之后有一个文件类型指示符。`"""``"""`

```
// Block string literal:
var block: String = """
    The winds grow high; so do your stomachs, lords.
    How irksome is this music to my heart!
    When such strings jar, what hope of harmony?
    I pray, my lords, let me compound this strife.
        -- History of Henry VI, Part II, Act II, Scene 1, W. Shakespeare
    """;
```

块字符串文本的终止行的缩进将从前面的所有行中删除。

字符串可能包含使用反斜杠 （） 引入的[转义序列](lexical_conventions/string_literals.md#escape-sequences)。[原始字符串文本](lexical_conventions/string_literals.md#raw-string-literals)可用于表示具有 s 和 s 的字符串。`\``\``"`

> 引用：
>
> - [字符串](lexical_conventions/string_literals.md)
> - 提案 [#199：字符串文本](https://github.com/carbon-language/carbon-lang/pull/199)

## 价值类别和价值阶段

每个值都有一个[值类别](https://en.wikipedia.org/wiki/Value_(computer_science)#lrvalue)，类似于[C++](https://en.cppreference.com/w/cpp/language/value_category)，即 *l 值*或 *r 值*。碳会自动将l值转换为r值，但不会在另一个方向上。

L 值具有存储和稳定的地址。它们可以被修改，假设它们的类型不是[`const`](#const)。

R 值可能没有专用存储。这意味着它们不能被修改，它们的地址通常不能被采用。R 值分为三种，称为*值阶段*：

- *常量*具有在编译时已知的值，并且该值在类型检查期间可用，例如用作数组的大小。其中包括文本（[整数](#integer-literals)、[浮点、](#floating-point-literals)[字符串](#string-literals)）、具体类型值（如或）、常量表达式和[`模板`参数](#checked-and-template-parameters)值。`f64``Optional(i32*)`
- *符号值*具有一个值，当[发生单态化](https://en.wikipedia.org/wiki/Monomorphization)时，该值将在编译的代码生成阶段已知，但在类型检查期间未知。这包括[已检查泛型参数](#checked-and-template-parameters)，以及带有已检查泛型参数的类型表达式，如 。`Optional(T*)`
- *运行时值*具有仅在运行时已知的动态值。

Carbon 会自动将常量转换为符号值，或任何值转换为运行时值：

```mermaid
constant
symbolic value
runtime value
l-value
```

常量转换为符号值和运行时值。如果对符号值执行检查符号值的操作，则符号值通常会转换为运行时值。如果运行时表达式的常量计算成功，则运行时值将转换为常量或符号值。

> **注意：**运行时值到其他阶段的转换是临时的，r 值的语义也是如此。请参阅待处理的提案 [#821：值、变量、指针和引用](https://github.com/carbon-language/carbon-lang/pull/821)。

## 复合类型

### 元组

元组是固定大小的值集合，可以具有不同的类型，其中每个值由其在元组中的位置标识。元组的一个示例用法是从函数返回多个值：

```
fn DoubleBoth(x: i32, y: i32) -> (i32, i32) {
  return (2 * x, 2 * y);
}
```

将此示例分解：

- 返回类型是两种类型的元组。`i32`
- 该表达式使用元组语法来生成包含两个值的元组。`i32`

这两个都是使用元组语法的表达式。唯一的区别是元组表达式的类型：一个是类型的元组，另一个是值的元组。换句话说，元组类型*是类型的元*组。`(<expression>, <expression>)`

元组的组件是按位置访问的，因此元素访问使用下标语法，但索引必须是编译时常量：

```
fn DoubleTuple(x: (i32, i32)) -> (i32, i32) {
  return (2 * x[0], 2 * x[1]);
}
```

元组类型是[结构化](https://en.wikipedia.org/wiki/Structural_type_system)的。

> **注意：**这是暂时的，尚未通过提案流程来设计元组。其中许多问题在被删除的提案[#111](https://github.com/carbon-language/carbon-lang/pull/111)中进行了讨论。

> 引用：[元组](tuples.md)

### 结构类型

碳还具有[结构类型](https://en.wikipedia.org/wiki/Structural_type_system)，其成员通过名称而不是位置来标识。这些称为*结构数据类*，也称为*结构类型*或*结构*。

结构类型和值都写在大括号 （...） 内。在这两种情况下，它们都有一个逗号分隔的成员列表，这些成员以句点 （） 开头，后跟字段名称。`{``}``.`

- 在结构类型中，字段名称后跟冒号 （） 和类型，如：中所示。`:``{.name: String, .count: i32}`
- 在结构值（称为*结构数据类文本*或*结构文本）*中，字段名称后跟等号 （） 和值，如 中所示。`=``{.key = "Joe", .count = 3}`

> 引用：
>
> - [结构类型](classes.md#struct-types)
> - 提案 [#561：基本类：用例、结构文字、结构类型和未来工作](https://github.com/carbon-language/carbon-lang/pull/561)
> - 提案 [#981：聚合的隐式转换](https://github.com/carbon-language/carbon-lang/pull/981)
> - 提案 [#710：数据类的默认比较](https://github.com/carbon-language/carbon-lang/issues/710)

### 指针类型

指向类型值的指针的类型写为 。碳指针不支持[指针算术](https://en.wikipedia.org/wiki/Pointer_(computer_programming));唯一的指针[操作](#expressions)是：`T``T*`

- 取消引用：给定一个指针，将值指向为 [l 值](#value-categories-and-value-phases)。 是 的句法糖。`p``*p``p``p->m``(*p).m`
- 地址：给定一个 [l 值](#value-categories-and-value-phases)，返回一个指向 的指针。`x``&x``x`

Carbon 中没有[空指针](https://en.wikipedia.org/wiki/Null_pointer)。要表示可能不引用有效对象的指针，请使用类型 。`Optional(T*)`

**待办事项：**也许Carbon将有[更严格的指针出处](https://www.ralfj.de/blog/2022/04/11/provenance-exposed.html)或对指针和整数之间的强制转换的限制。

> **注意：**虽然指针的语法已经确定，但指针的语义是临时的，可选的语法也是如此。请参阅待处理的提案 [#821：值、变量、指针和引用](https://github.com/carbon-language/carbon-lang/pull/821)。

> 引用：
>
> - 针对潜在客户的问题 [#520：我们是否应该使用对空格敏感的运算符固定性？](https://github.com/carbon-language/carbon-lang/issues/520)
> - 针对潜在客户的问题 [#523：对于指针类型，我们应该使用什么语法？](https://github.com/carbon-language/carbon-lang/issues/523)

### 数组和切片

包含 4 个值的数组的类型为 。只要元组的每个组件都可以[隐式转换为](expressions/implicit_conversions.md)目标元素类型，就会从元组隐式转换为相同长度的数组。在可以推断出数组大小的情况下，可以省略它，例如：`i32``[i32; 4]`

```
var i: i32 = 1;
// `[i32;]` equivalent to `[i32; 3]` here.
var a: [i32;] = (i, i, i);
```

可以使用方括号 （...） 访问数组的元素，如：`[``]``a[i]`

```
a[i] = 2;
Carbon.Print(a[0]);
```

> **待办事项：**片

> **注意：**这是临时的，尚未通过提案流程来设计数组。请参阅待处理的提案 [#1928：数组](https://github.com/carbon-language/carbon-lang/pull/1928)。

## 表达 式

表达式描述一些计算值。最简单的例子是一个文字数字，如：一个计算整数值42的表达式。`42`

Carbon 中的一些常见表达式包括：

- 文字：
  - [布尔值](#bool)： ，`true``false`
  - [整数](#integer-literals)： ，`42``-7`
  - [实数](#floating-point-literals)： ，`3.1419``6.022e+23`
  - [字符串](#string-literals)：`"Hello World!"`
  - [元组](#tuples)：`(1, 2, 3)`
  - [结构](#struct-types)：`{.word = "the", .count = 56}`
- [名称](#names)和[成员访问](expressions/member_access.md)
- [运营商](expressions#operators)：
  - [算术](expressions/arithmetic.md)： ， ， ， ， ， ，`-x``1 + 2``3 - 4``2 * 5``6 / 3``5 % 3`
  - [按位](expressions/bitwise.md)： 、 、 、`2 & 3``2 | 4``3 ^ 1``^7`
  - [位移](expressions/bitwise.md)位：，`1 << 3``8 >> 1`
  - [比较](expressions/comparison_operators.md)： ， ， ， ， ， ，`2 == 2``3 != 4``5 < 6``7 > 6``8 <= 8``8 >= 8`
  - [转换](expressions/as_expressions.md)：`2 as i32`
  - [逻辑](expressions/logical_operators.md)：、 、`a and b``c or d``not e`
  - [索引](#arrays-and-slices)：`a[3]`
  - [函数](#functions)调用：`f(4)`
  - [指针](#pointer-types)： 、 、`*p``p->m``&x`
  - [移动](#move)：`~x`
- [条件](expressions/if.md)：`if c then t else f`
- 括弧：`(7 + 8) * (3 - 1)`

当表达式出现在需要特定类型表达式的上下文中时，将应用[隐式转换](expressions/implicit_conversions.md)将表达式转换为目标类型。

> 引用：
>
> - [表达 式](expressions/)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)
> - 提案[#555：操作员优先顺序](https://github.com/carbon-language/carbon-lang/pull/555)
> - 提案[#601：操作员令牌](https://github.com/carbon-language/carbon-lang/pull/601)
> - 提案[#680：和，或者，不是](https://github.com/carbon-language/carbon-lang/pull/680)
> - 提案[#702：比较运算符](https://github.com/carbon-language/carbon-lang/pull/702)
> - 提案[#845：作为表达式](https://github.com/carbon-language/carbon-lang/pull/845)
> - 提案[#911：条件表达式](https://github.com/carbon-language/carbon-lang/pull/911)
> - 提案 [#1083：算术表达式](https://github.com/carbon-language/carbon-lang/pull/1083)

## 声明、定义和范围

*声明*引入一个新[名称](#names)，并说明该名称所代表的含义。对于某些类型的实体（如[函数](#functions)），有两种声明：*前向声明*和*定义*。对于这些实体，名称应该只有一个定义，最多应该有一个在定义名称之前引入名称的附加前向声明，以及[`match_first`块](generics/details.md#prioritization-rule)中任意数量的声明。前向声明可用于将接口与实现分开，例如在 [impl](#files-libraries-packages) [文件中定义的 api](#files-libraries-packages) 文件中声明名称。正向声明还允许在定义实体之前使用它们，例如允许循环引用。已声明但未定义的名称称为*不完整*，在某些情况下，对不完整名称可以执行的操作存在限制。在定义中，在到达定义末尾之前，定义的名称是不完整的，但在成员函数的主体中是完整的，因为它们[被解析为出现在定义之后](#class-functions-and-factory-functions)。

名称在最内层封闭[*作用域*](https://en.wikipedia.org/wiki/Scope_(computer_science))结束之前一直有效。有几种类型的作用域：

- 最外层的作用域，包括整个文件，
- 用大括号 （...） 括起来的作用域，以及`{``}`
- 包含单个声明的作用域。

例如，[函数](#functions)或[类](#classes)的参数名称在声明结束之前一直有效。函数或类本身的名称在封闭作用域结束之前一直可见。

> 引用：
>
> - [原理：信息积累]()
> - 提案[#875：原则：信息积累](https://github.com/carbon-language/carbon-lang/pull/875)
> - 潜在顾客问题 [#472：未解决的问题：调用稍后在同一文件中定义的函数](https://github.com/carbon-language/carbon-lang/issues/472)

## 模式

> **注意：**这是暂时的，尚未通过提案流程来设计模式。

*模式表示*如何接收一些正在匹配的数据。有两种模式：

- *可反驳*模式可能无法根据正在匹配的运行时值进行匹配。
- *无可辩驳的*模式保证匹配，只要代码类型检查。

在[简介](#tour-of-the-basics)中，[函数参数](#functions)、[变量 `var` 声明](#variable-var-declarations)和[常量`允许`声明](#constant-let-declarations)使用“名称类型”构造。这种结构是无可辩驳的模式的一个例子，事实上，任何无可辩驳的模式都可以用于这些立场。[`match` 语句](#match)可以包括可反驳模式和无可辩驳模式。`:`

> 引用：
>
> - [模式匹配](pattern_matching.md)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)

### 绑定模式

最常见的无可辩驳的模式是*绑定模式*，由新名称、冒号 （） 和类型组成。它将该类型的匹配值绑定到该名称。它只能匹配可能[隐式转换为](expressions/implicit_conversions.md)该类型的值。可以使用下划线 （） 代替名称来匹配值，但不要将任何名称绑定到该值。`:``_`

绑定模式默认为*`允许`绑定*。关键字用于使其成为 *`var` 绑定*。`var`

- 绑定的结果是名称绑定到 [r 值](#value-categories-and-value-phases)。这意味着该值无法修改，并且通常无法获取其地址。`let`
- 绑定具有专用存储，因此名称是可以修改并具有稳定地址的 [l 值](#value-categories-and-value-phases)。`var`

-binding 可能会触发原始值的副本，如果原始值是临时的，则可能触发移动，或者绑定可能是指向原始值的指针，如 [C++ 中的 `const` 引用](https://en.wikipedia.org/wiki/Reference_(C%2B%2B))。程序员不能观察到选择了这些选项中的哪一个（复制、移动或指针）。例如，Carbon 不允许在通过指针修改原始值时对其进行修改。此选择也可能受类型的影响。例如，不支持复制的类型将通过指针传递。`let`

[泛型绑定](#checked-and-template-parameters)使用冒号 （） 而不是冒号 （），并且只能匹配[常量或符号值](#value-categories-and-value-phases)，而不能匹配运行时值。`:!``:`

关键字可以代替绑定模式中的类型，只要该类型可以从同一声明中的值的类型推导出来。`auto`

### 解构模式

还有无可辩驳的*解构模式*，例如*元组解构*。元组解构模式看起来像一个模式元组。它只能用于匹配其组件与元组的组件模式匹配的元组值。一个示例用法是：

```
// `Bar()` returns a tuple consisting of an
// `i32` value and 2-tuple of `f32` values.
fn Bar() -> (i32, (f32, f32));

fn Foo() -> i64 {
  // Pattern in `var` declaration:
  var (p: i64, _: auto) = Bar();
  return p;
}
```

声明中使用的模式对 返回的元组值进行分解。第一个组件模式 ， 对应于 返回的值的第一个组件，其类型为 。这是允许的，因为存在从 到 的隐式转换。此转换的结果将分配给名称 。第二个分量模式 与 返回的值的第二个分量匹配，其类型为 。`var``Bar()``p: i64``Bar()``i32``i32``i64``p``_: auto``Bar()``(f32, f32)`

### 可反驳的模式

[`match` 语句](#match)中允许使用其他类型的模式，这些模式可能匹配，也可能不匹配，具体取决于表达式的运行时值：`match`

- *表达式模式*是一个表达式，例如 ，其值必须等于才能匹配。`42`
- *选项模式与选项*类型中的一个事例匹配，如[选项类型部分](#choice-types)所述。
- *动态强制转换模式*测试动态类型，如[继承](#inheritance)中所述。

有关可反驳模式的示例，请参阅[`匹配`](#match)。

> 引用：
>
> - [模式匹配](pattern_matching.md)
> - 潜在客户问题 [#1283：模式匹配和隐式转换应如何交互？](https://github.com/carbon-language/carbon-lang/issues/1283)

## 名称绑定声明

有两种类型的名称绑定声明：

- 常量声明，随 和 引入`let`
- 变量声明，随 引入。`var`

没有这些的前瞻性声明;所有名称绑定声明都是[定义](#declarations-definitions-and-scopes)。

### 常量声明`let`

声明将[无可辩驳的模式](#patterns)与值匹配。在此示例中，名称绑定到类型为 的值：`let``x``42``i64`

```
let x: i64 = 42;
```

下面是模式，后跟一个等号 （） 和要匹配的值 。[绑定模式](#binding-patterns)中的名称将引入到封闭[作用域](#declarations-definitions-and-scopes)中。`x: i64``=``42`

> **注：**声明是临时性的。请参阅待处理的提案 [#821：值、变量、指针和引用](https://github.com/carbon-language/carbon-lang/pull/821)。`let`

### 变量声明`var`

声明是相似的，除了绑定，所以下面是一个带有存储和地址的[l值](#value-categories-and-value-phases)，因此可以修改：`var``var``x`

```
var x: i64 = 42;
x = 7;
```

具有[未形成状态](#unformed-state)的类型的变量不需要在变量声明中初始化，但需要在使用之前进行赋值。

> 引用：
>
> - [变量](variables.md)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)
> - 提案[#257：内存和变量的初始化](https://github.com/carbon-language/carbon-lang/pull/257)
> - 提案 [#339：添加 `var   [ =  \];` 变量的语法](https://github.com/carbon-language/carbon-lang/pull/339)
> - 提案[#618：var排序](https://github.com/carbon-language/carbon-lang/pull/618)

### `auto`

如果 用作 或 声明中的类型，则该类型是初始值设定项表达式的静态类型，这是必需的。`auto``var``let`

```
var x: i64 = 2;
// The type of `y` is inferred to be `i64`.
let y: auto = x + 3;
// The type of `z` is inferred to be `bool`.
var z: auto = (y > 1);
```

> 引用：
>
> - [类型推断](type_inference.md)
> - 提案 [#851：vars 的自动关键字](https://github.com/carbon-language/carbon-lang/pull/851)

## 功能

功能是行为的核心单元。例如，这是一个函数[的前向声明](#declarations-definitions-and-scopes)，该函数添加两个 64 位整数：

```
fn Add(a: i64, b: i64) -> i64;
```

将其分解：

- `fn`是用于引入函数的关键字。
- 它的名字是 。这是添加到封闭[作用域](#declarations-definitions-and-scopes)的名称。`Add`
- 括号 （...） 中的[参数列表](#parameters)是以逗号分隔的[无可辩驳](#patterns)模式列表。`(``)`
- 它返回一个结果。不返回任何内容的函数会省略 和 返回类型。`i64``->`

您可以将此函数称为 .`Add(1, 2)`

函数定义是具有正文[块](#blocks-and-statements)而不是分号的函数声明：

```
fn Add(a: i64, b: i64) -> i64 {
  return a + b;
}
```

参数的名称在定义或声明结束之前一直处于作用域内。可以使用 省略 正向声明中的参数名称，但如果指定了，则必须与定义匹配。`_`

> 引用：
>
> - [功能](functions.md)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)
> - 提案 [#438：为函数声明添加语句语法](https://github.com/carbon-language/carbon-lang/pull/438)
> - 潜在顾客问题 [#476：可选参数名称（未使用的参数）](https://github.com/carbon-language/carbon-lang/issues/476)
> - 面向潜在客户的问题 [#1132：我们如何将前瞻声明与其定义相匹配？](https://github.com/carbon-language/carbon-lang/issues/1132)

### 参数

参数列表中的绑定默认为[`允许`绑定](#binding-patterns)，因此参数名称被视为 [r 值](#value-categories-and-value-phases)。这适用于输入参数。此绑定将使用指针实现，除非复制是合法的并且复制更便宜。

如果在绑定之前添加了关键字，则参数将被复制（或从临时存储移动到）到新存储，因此可以在函数体中进行更改。该副本可确保调用方看不到任何突变。`var`

使用[指针](#pointer-types)参数类型来表示[输入/输出参数](https://en.wikipedia.org/wiki/Parameter_(computer_programming)#Output_parameters)，允许函数修改调用方的变量。这使得这些修改的可能性可见：通过在调用方中使用地址，并在被调用方中使用取消引用。`&``*`

函数的输出应该优先返回。可以使用[元组](#tuples)或[结构](#struct-types)类型返回多个值。

> **注意：**参数传递的语义是临时的。请参阅待处理的提案 [#821：值、变量、指针和引用](https://github.com/carbon-language/carbon-lang/pull/821)。

### `auto`返回类型

如果用于代替返回类型，则从函数体推断函数的返回类型。它被设置为函数中[`返回`语句](#return)的参数的静态类型的[通用类型](#common-type)。这在前向声明中是不允许的。`auto`

```
// Return type is inferred to be `bool`, the type of `a > 0`.
fn Positive(a: i64) -> auto {
  return a > 0;
}
```

> 引用：
>
> - [类型推断](type_inference.md)
> - [函数返回子句](functions.md#return-clause)
> - 提案 [#826：函数返回类型推断](https://github.com/carbon-language/carbon-lang/pull/826)

### 块和语句

*块*是一系列*语句*。块定义[一个范围](#declarations-definitions-and-scopes)，并且与其他范围一样，用大括号 （...） 括起来。每个语句都以分号或块结尾。[表达式](#expressions)和 [`var`](#variable-var-declarations) 和 [`let`](#constant-let-declarations) 是有效的语句。`{``}`

块中的语句通常按照它们在源代码中出现的顺序执行，除非由 control-flow 语句修改。

函数的主体由块定义，某些[控制流语句](#control-flow)具有自己的代码块。它们嵌套在封闭范围内。例如，下面是一个函数定义，其中包含定义函数主体的语句块，以及一个作为语句一部分的嵌套块：`while`

```
fn Foo() {
  Bar();
  while (Baz()) {
    Quux();
  }
}
```

> 引用：
>
> - [块和语句](blocks_and_statements.md)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)

### 转让声明

赋值语句会改变赋值左侧描述的 [l 值](#value-categories-and-value-phases)的值。

- 分配：。 被分配了 值。`x = y;``x``y`
- 递增和递减：、 。 设置为 ，设置为 。`++i;``--j;``i``i + 1``j``j - 1`
- 复合赋值：、 、 、 、 、 、 、 。 等效于对于每个运算符 。`x += y;``x -= y;``x *= y;``x /= y;``x &= y;``x |= y;``x ^= y;``x <<= y;``x >>= y;``x @= y;``x = x @ y;``@`

与C++不同，这些赋值是语句，而不是表达式，并且不返回值。

> **注意：**赋值的语义是临时的。请参阅待处理的提案 [#821：值、变量、指针和引用](https://github.com/carbon-language/carbon-lang/pull/821)。

### 控制流

语句块通常按顺序执行。控制流语句为执行流和执行哪些语句提供了额外的控制。

某些控制流语句包含[块](#blocks-and-statements)。这些块将始终在大括号内....`{``}`

```
// Curly braces { ... } are required.
if (condition) {
  ExecutedWhenTrue();
} else {
  ExecutedWhenFalse();
}
```

这与C++不同，后者允许控制流构造省略单个语句周围的大括号。

> 引用：
>
> - [控制流](control_flow/README.md)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)
> - 提案[#623：需要大括号](https://github.com/carbon-language/carbon-lang/pull/623)

#### `if`和`else`

```
if`并提供语句的条件执行。语句包括：`else``if
```

- 一个介绍人，后跟括号中的条件。如果条件的计算结果为 ，则执行条件后面的块，否则将跳过该块。`if``true`
- 这后面可以跟零个或多个子句，如果所有先验条件的计算结果都为 ，则评估其条件，如果该评估为 到 ，则执行一个块。`else if``false``true`
- 最后一个可选子句，其中包含一个块，如果所有条件的计算结果均为 。`else``false`

例如：

```
if (fruit.IsYellow()) {
  Carbon.Print("Banana!");
} else if (fruit.IsOrange()) {
  Carbon.Print("Orange!");
} else {
  Carbon.Print("Vegetable!");
}
```

此代码将：

- 如果 为 ，则打印 。`Banana!``fruit.IsYellow()``true`
- 如果 是 和 则打印 。`Orange!``fruit.IsYellow()``false``fruit.IsOrange()``true`
- 如果上述两者都返回，请打印 。`Vegetable!``false`

> 引用：
>
> - [控制流](control_flow/conditionals.md)
> - 提案[#285：如果/否则](https://github.com/carbon-language/carbon-lang/pull/285)

#### 循环

> 参考：[循环](control_flow/loops.md)

##### `while`

```
while`语句循环，只要传递的表达式返回 。例如，这将打印 、 、 、 然后：`true``0``1``2``Done!
var x: i32 = 0;
while (x < 3) {
  Carbon.Print(x);
  ++x;
}
Carbon.Print("Done!");
```

> 引用：
>
> - [`而`循环](control_flow/loops.md#while)
> - 建议 [#340：添加类似C++`的 while` 循环](https://github.com/carbon-language/carbon-lang/pull/340)

##### `for`

```
for`语句支持基于范围的循环，通常基于容器。例如，这将打印 ：`String``names
for (var name: String in names) {
  Carbon.Print(name);
}
```

> 引用：
>
> - [`对于`循环](control_flow/loops.md#for)
> - 建议 [#353：`为`循环添加类似C++](https://github.com/carbon-language/carbon-lang/pull/353)

##### `break`

该语句会立即结束 or 循环。执行将从循环范围的末尾开始。例如，此过程将处理步骤，直到命中手动步骤（如果未命中手动步骤，则处理所有步骤）：`break``while``for`

```
for (var step: Step in steps) {
  if (step.IsManual()) {
    Carbon.Print("Reached manual step!");
    break;
  }
  step.Process();
}
```

> 引用：
>
> - [`break`](control_flow/loops.md#break)
> - 建议 [#340：添加类似C++`的 while` 循环](https://github.com/carbon-language/carbon-lang/pull/340)
> - 建议 [#353：`为`循环添加类似C++](https://github.com/carbon-language/carbon-lang/pull/353)

##### `continue`

该语句立即转到 或 的下一个循环。在 a 中，表达式继续执行。例如，这将打印文件的所有非空行，用于跳过空行：`continue``while``for``while``while``continue`

```
var f: File = OpenFile(path);
while (!f.EOF()) {
  var line: String = f.ReadLine();
  if (line.IsEmpty()) {
    continue;
  }
  Carbon.Print(line);
}
```

> 引用：
>
> - [`continue`](control_flow/loops.md#continue)
> - 建议 [#340：添加类似C++`的 while` 循环](https://github.com/carbon-language/carbon-lang/pull/340)
> - 建议 [#353：`为`循环添加类似C++](https://github.com/carbon-language/carbon-lang/pull/353)

#### `return`

该语句结束函数内的执行流，将执行返回给调用方。`return`

```
// Prints the integers 1 .. `n` and then
// returns to the caller.
fn PrintFirstN(n: i32) {
  var i: i32 = 0;
  while (true) {
    i += 1;
    if (i > n) {
      // None of the rest of the function is
      // executed after a `return`.
      return;
    }
    Carbon.Print(i);
  }
}
```

如果函数向调用方返回一个值，则该值由 return 语句中的表达式提供。例如：

```
fn Sign(i: i32) -> i32 {
  if (i > 0) {
    return 1;
  }
  if (i < 0) {
    return -1;
  }
  return 0;
}

Assert(Sign(-3) == -1);
```

> 引用：
>
> - [`return`](control_flow/return.md)
> - [`返回`语句](functions.md#return-statements)
> - 提案[#415：返回](https://github.com/carbon-language/carbon-lang/pull/415)
> - 提案[#538：返回时没有参数](https://github.com/carbon-language/carbon-lang/pull/538)

##### `returned var`

若要在返回变量时避免复制，请在变量的声明中添加前缀并使用，而不是返回表达式，如以下所示：`returned``return var`

```
fn MakeCircle(radius: i32) -> Circle {
  returned var c: Circle;
  c.radius = radius;
  // `return c` would be invalid because `returned` is in use.
  return var;
}
```

这代替[了C++的“命名返回值优化](https://en.wikipedia.org/wiki/Copy_elision#Return_value_optimization)”。

> 引用：
>
> - [`returned var`](control_flow/return.md#returned-var)
> - 提案[#257：内存和变量的初始化](https://github.com/carbon-language/carbon-lang/pull/257)

#### `match`

```
match`是类似于 C 和 C++ 的控制流，并镜像其他语言（如 Swift）中的类似构造。关键字后跟括号中的表达式，其值与声明匹配，每个声明按顺序包含[一个可反驳的模式](#refutable-patterns)。可反驳模式可以选择性地后跟一个表达式，该表达式可以使用模式中绑定的名称。`switch``match``case``if
```

执行第一个匹配的代码。可以在声明之后放置一个可选块，如果没有一个声明匹配，它将被执行。`case``default``case``case`

例如：`match`

```
fn Bar() -> (i32, (f32, f32));

fn Foo() -> f32 {
  match (Bar()) {
    case (42, (x: f32, y: f32)) => {
      return x - y;
    }
    case (p: i32, (x: f32, _: f32)) if (p < 13) => {
      return p * x;
    }
    case (p: i32, _: auto) if (p > 3) => {
      return p * Pi;
    }
    default => {
      return Pi;
    }
  }
}
```

> **注意：**这是暂时的，尚未通过提案流程进行声明设计。`match`

> 引用：
>
> - [模式匹配](pattern_matching.md)
> - 潜在客户问题 [#1283：模式匹配和隐式转换应如何交互？](https://github.com/carbon-language/carbon-lang/issues/1283)

## 用户定义类型

### 类

*名义类*或只是[*类*](https://en.wikipedia.org/wiki/Class_(computer_programming))是用户定义自己的[数据结构](https://en.wikipedia.org/wiki/Data_structure)或[记录类型的](https://en.wikipedia.org/wiki/Record_(computer_science))一种方式。

这是类[定义的](#declarations-definitions-and-scopes)一个示例：

```
class Widget {
  var x: i32;
  var y: i32;
  var payload: String;
}
```

将其分解：

- 这将定义一个名为 的类。 是添加到封闭[作用域](#declarations-definitions-and-scopes)的名称。`Widget``Widget`
- 名称后跟包含类*体*的大括号 （...），使其成为一个[定义](#declarations-definitions-and-scopes)。[前向声明](#declarations-definitions-and-scopes)将改为具有分号（）。`Widget``{``}``;`
- 这些大括号划定了类[的范围](#declarations-definitions-and-scopes)。
- 字段或[实例变量](https://en.wikipedia.org/wiki/Instance_variable)是使用 [`var` 声明](#variable-var-declarations)定义的。 有两个字段 （ 和 ） 和一个字段 （）。`Widget``i32``x``y``String``payload`

字段声明的顺序决定了字段的内存布局顺序。

类可能具有除在其作用域中声明的字段之外的其他类型的成员：

- [类函数](#class-functions-and-factory-functions)
- [方法](#methods)
- [`alias`](#aliases)
- [`让我们`](#constant-let-declarations)定义类常量。**待办事项：**另一种语法来定义与类关联的常量，如 或 ？`class let``static let`
- `class`，以定义[*成员类*或*嵌套类*](https://en.wikipedia.org/wiki/Inner_class)

在类的作用域内，非限定名可用于引用类本身。`Self`

类的成员是使用点 （） 表示法[访问](expressions/member_access.md)的，因此给定类型的实例 ，引用其字段。`.``dial``Widget``dial.payload``payload`

[结构数据类](#struct-types)和名义类都被视为*类类型*，但当它们不混淆时，它们通常分别称为“结构”和“类”。与结构一样，类通过名称引用其成员。与结构不同，类是[名义类型](https://en.wikipedia.org/wiki/Nominal_type_system#Nominal_typing)。

> 引用：
>
> - [类](classes.md#nominal-class-types)
> - 提案[#722：名义类和方法](https://github.com/carbon-language/carbon-lang/pull/722)
> - 提案 [#989：成员访问表达式](https://github.com/carbon-language/carbon-lang/pull/989)

#### 分配

在[有权访问](#access-control)所有类的字段的任何作用域中，结构[文本](#struct-types)和具有相同字段的类类型之间定义了[隐式转换](expressions/implicit_conversions.md)。这可用于使用类类型分配或初始化变量，如：

```
var sprocket: Widget = {.x = 3, .y = 4, .payload = "Sproing"};
sprocket = {.x = 2, .y = 1, .payload = "Bounce"};
```

> 引用：
>
> - [类：作业](classes.md#assignment)
> - 提案[#722：名义类和方法](https://github.com/carbon-language/carbon-lang/pull/722)
> - 提案 [#981：聚合的隐式转换](https://github.com/carbon-language/carbon-lang/pull/981)

#### 类函数和工厂函数

类还可以包含*类函数*。这些是作为类型成员访问的函数，如[C++中的静态成员函数](https://en.wikipedia.org/wiki/Method_(computer_programming)#Static_methods)，而不是作为实例成员的方法。它们通常用于定义创建实例的函数。碳不像C++那样有单独的[构造函数](https://en.wikipedia.org/wiki/Constructor_(object-oriented_programming))。

```
class Point {
  // Class function that instantiates `Point`.
  // `Self` in class scope means the class currently being defined.
  fn Origin() -> Self {
    return {.x = 0, .y = 0};
  }
  var x: i32;
  var y: i32;
}
```

请注意，如果在类作用域内提供了函数的定义，则将主体视为紧跟在最外层类定义之后定义。这意味着，即使诸如字段之类的成员在源中的声明晚于类函数，也会被视为已声明。

如果工厂函数中需要正在创建的实例的地址，则可以使用[`返回的 var` 功能](#returned-var)，例如：

```
class Registered {
  fn Make() -> Self {
    returned var result: Self = {...};
    StoreMyPointerSomewhere(&result);
    return var;
  }
}
```

此方法还可用于无法复制或移动的类型。

> 引用：
>
> - [所属分类： 建筑](classes.md#construction)
> - 提案[#722：名义类和方法](https://github.com/carbon-language/carbon-lang/pull/722)

#### 方法

类类型定义可以包括以下方法：

```
class Point {
  // Method defined inline
  fn Distance[me: Self](x2: i32, y2: i32) -> f32 {
    var dx: i32 = x2 - me.x;
    var dy: i32 = y2 - me.y;
    return Math.Sqrt(dx * dx + dy * dy);
  }
  // Mutating method declaration
  fn Offset[addr me: Self*](dx: i32, dy: i32);

  var x: i32;
  var y: i32;
}

// Out-of-line definition of method declared inline
fn Point.Offset[addr me: Self*](dx: i32, dy: i32) {
  me->x += dx;
  me->y += dy;
}

var origin: Point = {.x = 0, .y = 0};
Assert(Math.Abs(origin.Distance(3, 4) - 5.0) < 0.001);
origin.Offset(3, 4);
Assert(origin.Distance(3, 4) == 0.0);
```

这定义了一个具有两个整数数据成员和两个方法的类类型，并且：`Point``x``y``Distance``Offset`

- 方法被定义为类函数，参数在方括号内...在parens中的常规显式参数列表之前....`me``[``]``(``)`
- 使用成员语法调用方法，...和。。。。`origin.Distance(``)``origin.Offset(``)`
- `Distance`计算并返回到另一个点的距离，而不修改 .这在方法声明中使用表示。`Point``[me: Self]`
- `origin.Offset(`...修改 的值。这在方法声明中使用表示。因为调用这个方法需要取的地址，它只能在[非`const`](#const) [l值](#value-categories-and-value-phases)上调用。`)``origin``[addr me: Self*]``origin`
- 方法可以在词法上像 内联方式声明，也可以像 词法不内联一样声明。`Distance``Offset`

> 引用：
>
> - [方法](classes.md#methods)
> - 提案[#722：名义类和方法](https://github.com/carbon-language/carbon-lang/pull/722)

#### 遗产

Carbon中继承支持的理念是专注于继承是良好匹配的用例，并在其他情况下使用其他功能。例如，用于实现重用[的 mixins](#mixins) 和用于将接口与实现分离的[泛型](#generics)。这使得Carbon能够摆脱[多重继承，而多重继承](https://en.wikipedia.org/wiki/Multiple_inheritance)没有那么有效的实施策略。

默认情况下，类是[*最终类*](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)#Non-subclassable_classes)，这意味着它们可能不会扩展。类可以声明为允许使用 或 引入符而不是 扩展。是本身可能未实例化的基类。`base class``abstract class``class``abstract class`

```
base class MyBaseClass { ... }
```

可以*扩展*任一基类以获得*派生类*。派生类是最终类，除非它们本身已声明或 。类只能扩展单个类。碳仅支持单一继承，并将使用mixins而不是多重继承。`base``abstract`

```
base class MiddleDerived extends MyBaseClass { ... }
class FinalDerived extends MiddleDerived { ... }
// ❌ Forbidden: class Illegal extends FinalDerived { ... }
```

基类可以定义[虚拟方法](https://en.wikipedia.org/wiki/Virtual_function)。这些方法的实现可以在派生类中重写。默认情况下，方法是*非虚拟*的，虚拟方法的声明必须以以下三个关键字之一为前缀：

- 标记的方法在此类中具有定义，但在任何基中都没有定义。`virtual`
- 标记的方法在此类中没有定义，但必须在任何非派生类中具有定义。`abstract``abstract`
- 标记的方法在此类中具有定义，可重写基类中的任何定义。`impl`

指向派生类的指针可以强制转换为指向其基类之一的指针。通过指向基类的指针调用虚拟方法将使用派生类中提供的重写定义。具有方法的基类可以在 match 语句中使用[运行时类型信息](https://en.wikipedia.org/wiki/Run-time_type_information)来动态测试值的动态类型是否为某个派生类，如下所示：`virtual`

```
var base_ptr: MyBaseType* = ...;
match (base_ptr) {
  case dyn p: MiddleDerived* => { ... }
}
```

出于构造目的，派生类的行为就像使用其直接基类的类型调用其第一个字段一样。`base`

```
class MyDerivedType extends MyBaseType {
  fn Make() -> MyDerivedType {
    return {.base = MyBaseType.Make(), .derived_field = 7};
  }
  var derived_field: i32;
}
```

抽象类无法实例化，因此它们应该定义返回 的类函数。这些函数应标记为[`受保护，`](#access-control)以便它们只能由派生类使用。`partial Self`

```
abstract class AbstractClass {
  protected fn Make() -> partial Self {
    return {.field_1 = 3, .field_2 = 9};
  }
  // ...
  var field_1: i32;
  var field_2: i32;
}
// ❌ Error: can't instantiate abstract class
var abc: AbstractClass = ...;

class DerivedFromAbstract extends AbstractClass {
  fn Make() -> Self {
    // AbstractClass.Make() returns a
    // `partial AbstractClass` that can be used as
    // the `.base` member when constructing a value
    // of a derived class.
    return {.base = AbstractClass.Make(),
            .derived_field = 42 };
  }

  var derived_field: i32;
}
```

> 引用：
>
> - [类： 继承](classes.md#inheritance)
> - 提案[#777：继承](https://github.com/carbon-language/carbon-lang/pull/777)
> - 提案 [#820：隐式转换](https://github.com/carbon-language/carbon-lang/pull/820)

#### 存取控制

默认情况下，类成员可公开访问。可以将关键字前缀添加到成员的声明中，以将其限制为类的成员或任何朋友。或 方法可以在派生类中实现，即使它可能不被调用。`private``private virtual``private abstract`

可以使用命名现有函数或类型的类内的声明来声明好友。与C++不同，声明可能仅引用编译器可解析的名称，而不像前向声明那样起作用。`friend``friend`

```
protected`就像 ，但也提供了对派生类的访问。`private
```

> 引用：
>
> - [类成员的访问控制](classes.md#access-control)
> - 问题线索问题[#665：`私有`与`公共`*语法*策略，以及其他可见性工具，如`外部`/`api`/等。](https://github.com/carbon-language/carbon-lang/issues/665)
> - 提案[#777：继承](https://github.com/carbon-language/carbon-lang/pull/777)
> - 潜在顾客问题 [#971：公共 API 文件中的私有接口](https://github.com/carbon-language/carbon-lang/issues/971)

#### 析 构 函数

类的析构函数是在该类型的值的生存期结束时执行的自定义代码。它们使用关键字后跟 or（如[方法](#methods)所做的那样）和类定义中的代码块进行定义，如下所示：`destructor``[me: Self]``[addr me: Self*]`

```
class MyClass {
  destructor [me: Self] { ... }
}
```

艺术

```
class MyClass {
  // Can modify `me` in the body.
  destructor [addr me: Self*] { ... }
}
```

类的析构函数在其数据成员的析构函数之前运行。数据成员将按声明的相反顺序销毁。派生类在其基类之前被销毁。

抽象类或基类中的析构函数可以像[使用方法](#inheritance)一样声明。从具有虚拟析构函数的类派生的类中的析构函数必须使用关键字前缀进行声明。通过指向基类的指针删除派生类的实例是非法的，除非基类已声明或 。要在已知指向非抽象基类的指针不指向具有派生类型的值时删除该指针，请使用 。`virtual``impl``virtual``impl``UnsafeDelete`

> 引用：
>
> - [类： 析构函数](classes.md#destructors)
> - 提案 [#1154：析构函数](https://github.com/carbon-language/carbon-lang/pull/1154)

#### `const`

> **注意：**这是暂时的，尚未通过提案流程进行设计。`const`

对于每个类型，都有这样的类型：`MyClass``const MyClass`

- 数据表示形式是相同的，因此值可以隐式转换为 .`MyClass*``(const MyClass)*`
- [l 值](#value-categories-and-value-phases)可以自动转换为 r 值，与 l 值可以自动转换为 r 值的方式相同。`const MyClass``MyClass``MyClass`
- 如果 的成员具有类型，则 的成员具有 类型 。`x``MyClass``T``x``const MyClass``const T`
- 的 API 是 的子集，不包括 所有采用 的方法。`const MyClass``MyClass``[addr me: Self*]`

请注意，对于形成指针类型，绑定比后缀 -更紧密，因此等于 。`const``*``const MyClass*``(const MyClass)*`

此示例使用[“方法”部分中](#methods)的定义：`Point`

```
var origin: Point = {.x = 0, .y = 0};

// ✅ Allowed conversion from `Point*` to
// `const Point*`:
let p: const Point* = &origin;

// ✅ Allowed conversion of `const Point` l-value
// to `Point` r-value.
let five: f32 = p->Distance(3, 4);

// ❌ Error: mutating method `Offset` excluded
// from `const Point` API.
p->Offset(3, 4);

// ❌ Error: mutating method `AssignAdd.Op`
// excluded from `const i32` API.
p->x += 2;
```

#### 未形成状态

类型指示它们通过[实现特定接口](#interfaces-and-implementations)来支持未格式化状态，否则在声明该类型的变量时必须显式初始化它们。

对象的未形成状态是满足以下属性的状态：

- 使用类型的正常赋值实现，从完全形成的值进行赋值是正确的。
- 销毁必须使用类型的正常销毁实现正确。
- 销毁必须是可选的。无论析构函数是否针对未形成的对象运行，包括不泄漏资源，程序的行为都必须是等效的。

对于未形成状态，一个类型可能具有多个内存中表示形式，并且这些表示形式可能与该类型的有效完全格式值相同。例如，所有值都是任何类型中未形成状态的法律表示，具有琐碎析构函数（如 ）。类型可以为[强化生成模式](#build-modes)定义其他初始化。例如，这会导致在此模式下处于未格式化状态时将整数设置为。`i32``0`

除了从完全形成的值进行析构或赋*值之外，*对未形成对象的任何操作都是错误，即使其内存中的表示形式是该类型的有效值。

> 引用：
>
> - 提案[#257：内存和变量的初始化](https://github.com/carbon-language/carbon-lang/pull/257)

#### 移动

碳将允许类型定义它们是否以及如何移动。从函数返回值或使用 *move 运算符* 返回值时，可能会发生这种情况。这将保持[未成形状态并](#unformed-state)返回其旧值。`~x``x`

> **注意：**这是暂时的。移动运算符已讨论过，但在已接受的提案 [#257：内存和变量的初始化](https://github.com/carbon-language/carbon-lang/pull/257)中未提出。请参阅待处理的提案 [#821：值、变量、指针和引用](https://github.com/carbon-language/carbon-lang/pull/821)。

#### 米辛斯

与[继承](#inheritance)相比，Mixins允许以不同的权衡进行重用。Mixins专注于实现重用，例如可以使用[CRTP](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern)或C++中的[多重继承](https://en.wikipedia.org/wiki/Multiple_inheritance)来完成。

> **待办事项：**mixins的设计仍在开发中。此处的详细信息是临时的。mixin 用例包含在已接受的提案 [#561 中：基本类：用例、结构文字、结构类型和未来工作](https://github.com/carbon-language/carbon-lang/pull/561)。

### 选择类型

*选择类型*是一个[标记的联合，](https://en.wikipedia.org/wiki/Tagged_union)它可以将不同类型的数据存储在可以容纳最大数据的存储空间中。选项类型具有名称和以逗号 （） 分隔的事例列表。每个事例都有一个名称和一个可选参数列表。`,`

```
choice IntResult {
  Success(value: i32),
  Failure(error: String),
  Cancelled
}
```

选项类型的值是其中一种情况，加上该情况的参数值（如果有）。可以通过命名事例并为参数提供值（如果有）来构造值：

```
fn ParseAsInt(s: String) -> IntResult {
  var r: i32 = 0;
  for (c: i32 in s) {
    if (not IsDigit(c)) {
      // Equivalent to `IntResult.Failure(...)`
      return .Failure("Invalid character");
    }
    // ...
  }
  return .Success(r);
}
```

可以使用 [`match` 语句](#match)使用选项类型值：

```
match (ParseAsInt(s)) {
  case .Success(value: i32) => {
    return value;
  }
  case .Failure(error: String) => {
    Display(error);
  }
  case .Cancelled => {
    Terminate();
  }
}
```

如果没有其他数据与选项相关联，它们还可以表示[枚举类型](https://en.wikipedia.org/wiki/Enumerated_type)，如：

```
choice LikeABoolean { False, True }
```

> 引用：
>
> - 提案 [#157：求和类型的设计方向](https://github.com/carbon-language/carbon-lang/pull/157)
> - 提案[#162：基本语法](https://github.com/carbon-language/carbon-lang/pull/162)

## 名字

名称由[声明引入，](#declarations-definitions-and-scopes)在它们出现的范围结束之前一直有效。代码在源中引用的名称不得早于声明的名称。在可执行作用域（如函数体）中，找不到稍后声明的名称。在声明性作用域（如包、类和接口）中，引用稍后声明的名称是错误的，只是内联类成员函数体[被分析为好像它们出现在类之后](#class-functions-and-factory-functions)。

Carbon 中的名称由一系列字母、数字和下划线组成，并以字母开头。我们打算遵循 [Unicode 的附录 31](https://unicode.org/reports/tr31/) 来选择有效的标识符字符，但尚未选择一组具体的有效字符。

> 引用：
>
> - [词汇约定](lexical_conventions)
> - [原理：信息积累]()
> - 提案 [#142：Unicode 源文件](https://github.com/carbon-language/carbon-lang/pull/142)
> - 潜在顾客问题 [#472：未解决的问题：调用稍后在同一文件中定义的函数](https://github.com/carbon-language/carbon-lang/issues/472)
> - 提案[#875：原则：信息积累](https://github.com/carbon-language/carbon-lang/pull/875)

### 文件、库、包

- **文件**被分组到库中，这些库又被分组到包中。
- **库**是通过导入重用代码的粒度。
- **包**是分发的单位。

每个库必须只有一个文件。此文件包括库的所有公共名称的声明。这些声明的定义必须位于库中的某个文件（文件或文件）中。`api``api``impl`

每个包都有自己的命名空间。这意味着包中的库需要协调以避免名称冲突，但不能跨包进行协调。

> 引用：
>
> - [代码和名称组织](code_and_name_organization)
> - 提案[#107：代码和名称组织](https://github.com/carbon-language/carbon-lang/pull/107)

### 包装声明

文件以可选的包声明开头，包括：

- 关键字介绍人，`package`
- 指定包名称的可选标识符，
- 可选，后跟带有库名称的字符串，`library`
- 或 、 和`api``impl`
- 终止分号 （）。`;`

例如：

```
// Package name is `Geometry`.
// Library name is "Shapes".
// This file is an `api` file, not an `impl` file.
package Geometry library "Shapes" api;
```

本声明的部分内容可以省略：

- 如果省略包名称，如 中所示，则该文件将参与默认包。任何其他包都不能从默认包导入。`package library "Main" api;`
- 如果未指定库关键字（如 中所示），则此文件将参与默认库。`package Geometry api;`
- 如果文件根本没有包声明，则它是属于默认包和默认库的文件。这尤其适用于测试和较小的示例。任何其他库都不能从默认包中导入此库。可以使用包声明将其拆分到多个文件中。`api``impl``package impl;`

程序不需要使用默认包，但如果使用，则应包含入口点函数。默认情况下，入口点函数来自默认包。`Run`

> 引用：
>
> - [代码和名称组织](code_and_name_organization)
> - 提案[#107：代码和名称组织](https://github.com/carbon-language/carbon-lang/pull/107)

### 进口

在包声明之后，文件可能包括声明。其中包括包名称和库名称（可选）。如果省略该库，则导入该包的默认库。`import``library`

```
// Import the "Vector" library from the
// `LinearAlgebra` package.
import LinearAlgebra library "Vector";
// Import the default library from the
// `ArbitraryPrecision` package.
import ArbitraryPrecision;
```

该语法将该名称作为命名给定包的[`私有`](#name-visibility)名称引入。它不能用于导入当前包的库。从该包中导入其他库会使 的其他成员可见。`import PackageName ...``PackageName``PackageName`

通过省略包名称来导入当前包中的库。

```
// Import the "Vertex" library from the same package.
import library "Vertex";
// Import the default library from the same package.
import library default;
```

该语法将给定库中的所有公共顶级名称作为[`私有`](#name-visibility)名称添加到当前文件的顶级范围，对于[命名空间](#namespaces)中的名称也是如此。`import library ...`

每个文件都会自动导入其库的文件。`impl``api`

所有声明都必须出现在文件中所有其他非声明之前。`import``package`

> 引用：
>
> - [代码和名称组织](code_and_name_organization)
> - 提案[#107：代码和名称组织](https://github.com/carbon-language/carbon-lang/pull/107)

### 名称可见性

从导入的库中可见的名称由以下规则确定：

- 默认情况下，文件中的声明是 *public* 的，这意味着导入该库的任何文件都可见。这与类成员匹配，这些类成员也是[默认的公共](#access-control)成员。`api`
- 文件中声明的前缀使名称*库成为私有*的。这意味着该名称在文件和同一库的所有文件中可见。`private``api``impl`
- 名称的可见性由其第一个声明确定，在文件之前考虑文件。前缀只允许在第一个声明上使用。`api``impl``private`
- 在文件中声明的名称而不是相应的文件是*文件私有*的，这意味着仅在该文件中可见。其第一个声明必须用前缀标记。**待办事项：**这需要在一项提案中最终确定，以解决[#665](https://github.com/carbon-language/carbon-lang/issues/665#issuecomment-914661914)和[#1136](https://github.com/carbon-language/carbon-lang/issues/1136)之间的不一致。`impl``api``private`
- 私有名称不会与它们所私有区域之外的名称冲突：两个不同的库可以具有不同的私有名称而不会发生冲突，但私有名称与同一范围内的公共名称冲突。`foo`

在程序中可传递使用的包中，最多只能将给定名称声明为 public。`api`

> 引用：
>
> - [从 API 文件导出实体](code_and_name_organization/README.md#exporting-entities-from-an-api-file)
> - 问题线索问题[#665：`私有`与`公共`*语法*策略，以及其他可见性工具，如`外部`/`api`/等。](https://github.com/carbon-language/carbon-lang/issues/665)
> - 提案 [#752：API 文件默认公开](https://github.com/carbon-language/carbon-lang/pull/752)
> - 提案 [#931：通用 impls 访问（细节 4）](https://github.com/carbon-language/carbon-lang/pull/931)
> - 针对潜在客户的问题 [#1136：源文件中的顶级范围是什么，其中有哪些名称？](https://github.com/carbon-language/carbon-lang/issues/1136)

### 包范围

文件中的顶级作用域是包的作用域。这意味着：

- 在此作用域（及其子命名空间）中，将显示同一包中的所有可见名称。这包括来自同一文件的名称、文件内库文件中的名称以及来自同一包的导入库的名称。`api``impl`
- 在包成员的名称可能与其他内容发生冲突的作用域中，语法可用于命名当前包的成员。`package.Foo``Foo`

在此示例中，名称 和 在一个作用域中使用，其中它们可能意味着两个不同的东西，并且[需要限定条件来消除歧义](#name-lookup)：`F``P`

```
import P;
fn F();
class C {
  fn F();
  class P {
    fn H();
  }
  fn G() {
    // ❌ Error: ambiguous whether `F` means
    // `package.F` or `package.C.F`.
    F();
   // ✅ Allowed: fully qualified
    package.F();
    package.C.F();
    // ✅ Allowed: unambiguous
    C.F();
    // ❌ Error: ambiguous whether `P` means
    // `package.P` or `package.C.P`.
    P.H();
    // ✅ Allowed
    package.P.H();
    package.C.P.H();
    C.P.H();
  }
}
```

> 引用：
>
> - [代码和名称组织](code_and_name_organization)
> - 提案[#107：代码和名称组织](https://github.com/carbon-language/carbon-lang/pull/107)
> - 提案 [#752：API 文件默认公开](https://github.com/carbon-language/carbon-lang/pull/752)
> - 针对潜在客户的问题 [#1136：源文件中的顶级范围是什么，其中有哪些名称？](https://github.com/carbon-language/carbon-lang/issues/1136)

### 命名空间

声明定义了一个名称，该名称可用作以后声明的名称的前缀。定义命名空间的成员时，该命名空间的其他成员将被视为作用域中，并且可以通过名称[查找](#name-lookup)找到，而无需命名空间前缀。在此示例中，包在命名空间中定义了它的一些成员：`namespace``P``N`

```
package P api;

// Defines namespace `N` within the current package.
namespace N;

// Defines namespaces `M` and `M.L`.
namespace M.L;

fn F();
// ✅ Allowed: Declares function `G` in namespace `N`.
private fn N.G();
// ❌ Error: `Bad` hasn't been declared.
fn Bad.H();

fn J() {
  // ❌ Error: No `package.G`
  G();
}

fn N.K() {
  // ✅ Allowed: Looks in both `package` and `package.N`.
  // Finds `package.F` and `package.N.G`.
  F();
  G();
}

// ✅ Allowed: Declares function `R` in namespace `M.L`.
fn M.L.R();
// ✅ Allowed: Declares function `Q` in namespace `M`.
fn M.Q();
```

另一个包导入可以通过在包名称后跟命名空间前加上前缀来引用该命名空间的公共成员：`P``P`

```
import P;

// ✅ Allowed: `F` is public member of `P`.
P.F();
// ❌ Error: `N.G` is a private member of `P`.
P.N.G();
// ✅ Allowed: `N.K` is public member of `P`.
P.N.K();
// ✅ Allowed: `M.L.R` is public member of `P`.
P.M.L.R();
// ✅ Allowed: `M.Q` is public member of `P`.
P.M.Q();
```

> 引用：
>
> - [“代码和名称组织”中的“命名空间”](code_and_name_organization/README.md#namespaces)
> - [“限定名称和成员访问权限”中的“包和命名空间成员”](expressions/member_access.md#package-and-namespace-members)
> - 提案[#107：代码和名称组织](https://github.com/carbon-language/carbon-lang/pull/107)
> - 提案 [#989：成员访问表达式](https://github.com/carbon-language/carbon-lang/pull/989)
> - 针对潜在客户的问题 [#1136：源文件中的顶级范围是什么，其中有哪些名称？](https://github.com/carbon-language/carbon-lang/issues/1136)

### 命名约定

我们的命名约定是：

- 对于惯用碳代码：
  - `UpperCamelCase`当命名实体不能具有动态变化的值时，将使用。例如，函数、命名空间或编译时常量值。请注意，[`虚拟`方法](#inheritance)的命名方式与与其他函数和方法一致。
  - `lower_snake_case`当命名实体的值在运行时之前未知时（例如，对于变量），将使用。
- 对于碳提供的功能：
  - 关键字和类型文本将使用 。`lower_snake_case`
  - 其他代码将使用惯用碳代码的约定。

> 引用：
>
> - [命名约定](naming_conventions.md)
> - 提案[#861：命名约定](https://github.com/carbon-language/carbon-lang/pull/861)

### 别名

`alias`将一个名称声明为等效于另一个名称，例如：

```
alias NewName = SomePackage.OldName;
```

请注意，等号 （） 的右侧是名称而不是值，因此不允许这样做。这允许使用命名空间等实体，这些实体不是 Carbon 中的值。`=``alias four = 4;``alias`

这可以在增量迁移期间更改名称时使用。例如，将允许您在旧名称和新名称之间迁移客户端时，类中的数据字段具有两个名称。`alias`

```
class MyClass {
  var new_name: String;
  alias old_name = new_name;
}

var x: MyClass = {.new_name = "hello"};
Carbon.Assert(x.old_name == "hello");
```

另一个用途是在公共 API 中包含名称。例如，可用于将接口实现中的名称作为类或[命名约束](generics/details.md#named-constraints)的成员包括在内，可能已重命名：`alias`

```
class ContactInfo {
  external impl as Printable;
  external impl as ToPrinterDevice;
  alias PrintToScreen = Printable.Print;
  alias PrintToPrinter = ToPrinterDevice.Print;
  ...
}
```

> 引用：
>
> - [别名](aliases.md)
> - [“代码和名称组织”中的“别名”](code_and_name_organization/README.md#aliasing)
> - [`别名` 来自外部 impl 的名称](generics/details.md#external-impl)
> - [`别名` 命名约束中的名称](generics/details.md#named-constraints)
> - 提案[#107：代码和名称组织](https://github.com/carbon-language/carbon-lang/pull/107)
> - 提案 [#553：泛型详述第 1 部分](https://github.com/carbon-language/carbon-lang/pull/553)
> - 潜在顾客问题 [#749：别名语法](https://github.com/carbon-language/carbon-lang/issues/749)
> - 提案 [#989：成员访问表达式](https://github.com/carbon-language/carbon-lang/pull/989)

### 名称查找

Carbon 名称查找的一般原则是，我们在所有相关范围内查找名称，如果发现名称引用多个不同的实体，则会报告错误。因此，Carbon需要通过添加限定符来消除歧义，而不是对名称进行任何[阴影](https://en.wikipedia.org/wiki/Variable_shadowing)处理。有关示例，请参阅[“包范围”部分](#package-scope)。

非限定名称查找遍历语义封闭的作用域，而不仅仅是词法封闭作用域。因此，当在 中执行查找时，我们将查看 、 、 和包范围，即使词法封闭范围是包范围。这意味着方法的定义将在类的作用域中查找名称，即使它在词法上写得不合时宜：`fn MyNamespace.MyClass.MyNestedClass.MyFunction()``MyNestedClass``MyClass``MyNamespace`

```
class C {
  fn F();
  fn G();
}
fn C.G() {
  // ✅ Allowed: resolves to `package.C.F`.
  F();
}
```

[成员名称查找](expressions/member_access.md)遵循类似的理念。如果已知[已检验泛型类型参数](#checked-and-template-parameters)由于使用 [`&`](#combining-constraints) 或 [`where` 子句](generics/details.md#where-constraints)的约束而实现多个接口，则对该类型的成员名称查找将在所有接口中查找。如果以多个形式找到该名称，则必须通过使用复合成员访问 （[1](expressions/member_access.md)， [2](generics/details.md#qualified-member-names-and-compound-member-access)） 进行限定来消除歧义。除了约束之外，[模板泛型类型参数](#checked-and-template-parameters)还执行对调用方类型的查找。

Carbon还拒绝了如果文件中的所有声明（包括后来出现的声明）在任何地方都可见的案例，而不仅仅是在它们出现点之后，那么这些案例将是无效的：

```
class C {
  fn F();
  fn G();
}
fn C.G() {
  F();
}
// Error: use of `F` in `C.G` would be ambiguous
// if this declaration was earlier.
fn F();
```

> 引用：
>
> - [名称查找](name_lookup.md)
> - [“表达式”的“限定名称和成员访问”部分](expressions/README.md#qualified-names-and-member-access)
> - [限定名称和成员访问权限](expressions/member_access.md)
> - [原理：信息积累]()
> - 提案[#875：原则：信息积累](https://github.com/carbon-language/carbon-lang/pull/875)
> - 提案 [#989：成员访问表达式](https://github.com/carbon-language/carbon-lang/pull/989)
> - 针对潜在客户的问题 [#1136：源文件中的顶级范围是什么，其中有哪些名称？](https://github.com/carbon-language/carbon-lang/issues/1136)

#### 常见类型的名称查找

我们希望普遍使用的常见类型将为每个文件提供，就好像有一个特殊的“prelude”包自动导入到每个文件中一样。专用类型文字语法，例如并引用此包中定义的类型，基于[“所有 API 都是库 API”原则]()。`api``i32``bool`

> **待办事项：**Prelude暂时导入包含常见设施（如）和操作员过载接口的软件包。`Carbon``Print`

> 引用：
>
> - [名称查找](name_lookup.md)
> - [原则：所有 API 都是库 API]()
> - 潜在客户问题 [#750：碳提供要素的命名约定](https://github.com/carbon-language/carbon-lang/issues/750)
> - 潜在顾客问题 [#1058：核心功能的接口应如何命名？](https://github.com/carbon-language/carbon-lang/issues/1058)
> - 提案 [#1280：原则：所有 API 都是库 API](https://github.com/carbon-language/carbon-lang/pull/1280)

## 泛 型

泛型允许使用编译时参数编写[函数](#functions)和[类](#classes)等Carbon构造，并使用这些参数泛型应用于不同类型的类型。例如，此函数具有一个 type 参数，该参数可以是实现接口的任何类型。`Min``T``Ordered`

```
fn Min[T:! Ordered](x: T, y: T) -> T {
  // Can compare `x` and `y` since they have
  // type `T` known to implement `Ordered`.
  return if x <= y then x else y;
}

var a: i32 = 1;
var b: i32 = 2;
// `T` is deduced to be `i32`
Assert(Min(a, b) == 1);
// `T` is deduced to be `String`
Assert(Min("abc", "xyz") == "abc");
```

由于类型参数位于演绎参数列表中的方括号 （...） 中，在括号 （...） 中的显式参数列表之前，因此 的值是根据显式参数的类型确定的，而不是作为单独的显式参数传递的。`T``[``]``(``)``T`

> 参考资料：**待办事项：**重温
>
> - [泛型：概述](generics/overview.md)
> - 提案[#524：泛型概述](https://github.com/carbon-language/carbon-lang/pull/524)
> - 提案 [#553：泛型详述第 1 部分](https://github.com/carbon-language/carbon-lang/pull/553)
> - 提案 [#950：通用详细信息 6：删除分面](https://github.com/carbon-language/carbon-lang/pull/950)

### 已检查的参数和模板参数

指示 是在编译时传递*的已检查*参数。这里的“Checked”表示在定义函数时对 的主体进行类型检查，与特定的类型值无关，并且名称查找被委托给 （在本例中）的约束。此类型检查等效于说函数将在给定实现接口的任何类型的情况下通过类型检查。然后调用只需要检查推导出的类型值来实现。`:!``T``Min``T``T``Ordered``T``Ordered``Min``T``Ordered`

也可以通过将参数以关键字作为前缀来声明该参数为*模板*参数，如 中所示。`template``template T:! Type`

```
fn Convert[template T:! Type](source: T, template U:! Type) -> U {
  var converted: U = source;
  return converted;
}

fn Foo(i: i32) -> f32 {
  // Instantiates with the `T` implicit argument set to `i32` and the `U`
  // explicit argument set to `f32`, then calls with the runtime value `i`.
  return Convert(i, f32);
}
```

碳模板遵循与[C++模板](https://en.wikipedia.org/wiki/Template_(C%2B%2B))相同的基本范例：它们在调用时实例化，从而导致延迟类型检查，鸭子键入和延迟绑定。

与C++模板的一个区别是，Carbon 模板实例化不受 SFINAE C++规则 （[1](https://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error)， [2](https://en.cppreference.com/w/cpp/language/sfinae)） 的控制，而是由编译时评估的显式子句控制。该子句位于声明的末尾，并且条件只能使用类型检查时已知的常量值，包括参数。`if``if``template`

```
class Array(template T:! Type, template N:! i64)
    if N >= 0 and N < MaxArraySize / sizeof(T);
```

除了任何约束之外，对模板类型参数的成员查找都是在调用方提供的实际类型值*中*完成的。这意味着在使用特定具体类型实例化模板之前，无法完成依赖于模板[参数的任何内容](generics/terminology.md#dependent-names)的成员名称查找和类型检查。当约束为正值时，这给出了类似于C++模板的语义。然后可以增量方式添加约束，编译器验证语义是否保持不变。添加完所有约束后，删除切换到已检查参数的单词是安全的。`Type``template`

已检查参数[的值阶段](#value-categories-and-value-phases)是符号值，而模板参数的值阶段是常量。

尽管通常首选已检查的泛型，但模板允许在C++和Carbon之间转换代码，并解决泛型的类型检查严谨性有问题的一些情况。

> 引用：
>
> - [模板](templates.md)
> - 提案 [#553：泛型详述第 1 部分](https://github.com/carbon-language/carbon-lang/pull/553)
> - 潜在顾客问题 [#949：受约束的模板名称查找](https://github.com/carbon-language/carbon-lang/issues/949)
> - 提案 [#989：成员访问表达式](https://github.com/carbon-language/carbon-lang/pull/989)

### 接口和实现

*接口*指定类型可能满足的一组要求。接口既充当对调用方可能提供的类型的约束，也充当可能假定满足该约束的类型的功能。

```
interface Printable {
  // Inside an interface definition `Self` means
  // "the type implementing this interface".
  fn Print[me: Self]();
}
```

除了功能要求之外，接口还可以包含：

- [要求实现其他接口](generics/details.md#interface-requiring-other-interfaces)或[此接口扩展的接口](generics/details.md#interface-extension)
- [关联的类型](generics/details.md#associated-types)和其他[关联的常量](generics/details.md#associated-constants)
- [接口默认值](generics/details.md#interface-defaults)
- [`最终`接口成员](generics/details.md#final-members)

类型仅当存在显式声明时才实现接口。仅仅拥有具有正确签名的函数是不够的。`impl``Print`

```
class Circle {
  var radius: f32;

  impl as Printable {
    fn Print[me: Self]() {
      Carbon.Print("Circle with radius: {0}", me.radius);
    }
  }
}
```

在本例中，是 的成员。接口也可以[在外部](generics/details.md#external-impl)实现，这意味着接口的成员不是该类型的直接成员。仍可以使用复合成员访问语法 （[1](expressions/member_access.md)， [2](generics/details.md#qualified-member-names-and-compound-member-access)） 来限定成员的名称来调用这些方法，如 中所示。外部实现不必与类型定义位于同一库中，受孤儿规则 （[1](generics/details.md#impl-lookup)， [2](generics/details.md#orphan-rule)） 的约束以实现[一致性](generics/terminology.md#coherence)。`Print``Circle``x.(Printable.Print)()`

接口和实现可以通过将大括号 （...） 中的定义范围替换为分号来[前向声明](generics/details.md#forward-declarations-and-cyclic-references)。`{``}`

> 引用：
>
> - [泛型：接口](generics/details.md#interfaces)
> - [泛型：实现接口](generics/details.md#implementing-interfaces)
> - 提案 [#553：泛型详述第 1 部分](https://github.com/carbon-language/carbon-lang/pull/553)
> - 提案 [#731：泛型详细信息 2：适配器、关联类型、参数化接口](https://github.com/carbon-language/carbon-lang/pull/731)
> - 提案[#624：一致性：术语，理由，考虑的替代方案](https://github.com/carbon-language/carbon-lang/pull/624)
> - 提案 [#990：泛型详细信息 8：接口默认成员和最终成员](https://github.com/carbon-language/carbon-lang/pull/990)
> - 提案 [#1084：泛型细节 9：前向声明](https://github.com/carbon-language/carbon-lang/pull/1084)
> - 面向潜在客户的问题 [#1132：我们如何将前瞻声明与其定义相匹配？](https://github.com/carbon-language/carbon-lang/issues/1132)

### 组合约束

函数可能需要调用类型通过使用 & 符号 （） 组合它们来实现多个接口：`&`

```
fn PrintMin[T:! Ordered & Printable](x: T, y: T) {
  // Can compare since type `T` implements `Ordered`.
  if (x <= y) {
    // Can call `Print` since type `T` implements `Printable`.
    x.Print();
  } else {
    y.Print();
  }
}
```

函数体可以调用任一接口中的函数，但作为两者成员的名称除外。在这种情况下，请使用复合成员访问语法 （[1](expressions/member_access.md)， [2](generics/details.md#qualified-member-names-and-compound-member-access)） 限定成员的名称，如下所示：

```
fn DrawTies[T:! Renderable & GameResult](x: T) {
  if (x.(GameResult.Draw)()) {
    x.(Renderable.Draw)();
  }
}
```

> 引用：
>
> - [通过类型类型和类型组合接口](generics/details.md#combining-interfaces-by-anding-type-of-types)
> - 问题 -for-leads 问题 [#531：将接口与 `+` or `&`](https://github.com/carbon-language/carbon-lang/issues/531)
> - 提案 [#553：泛型详述第 1 部分](https://github.com/carbon-language/carbon-lang/pull/553)

### 关联类型

关联类型是接口的类型成员，其值由该接口针对特定类型的实现确定。这些值在实现中设置为编译时值，因此请在没有初始值设定项[的 `let` 声明](#constant-let-declarations)中使用 [`：！` 泛型语法](#checked-and-template-parameters)。这允许接口中函数签名的类型发生变化。例如，描述[堆栈](https://en.wikipedia.org/wiki/Stack_(abstract_data_type))的接口可能使用关联类型来表示堆栈中存储的元素的类型。

```
interface StackInterface {
  let ElementType:! Movable;
  fn Push[addr me: Self*](value: ElementType);
  fn Pop[addr me: Self*]() -> ElementType;
  fn IsEmpty[addr me: Self*]() -> bool;
}
```

然后，实现的不同类型可以使用子句为接口成员指定不同的类型值：`StackInterface``ElementType``where`

```
class IntStack {
  impl as StackInterface where .ElementType == i32 {
    fn Push[addr me: Self*](value: i32);
    // ...
  }
}

class FruitStack {
  impl as StackInterface where .ElementType == Fruit {
    fn Push[addr me: Self*](value: Fruit);
    // ...
  }
}
```

> 引用：
>
> - [泛型：关联类型](generics/details.md#associated-types)
> - 提案 [#731：泛型详细信息 2：适配器、关联类型、参数化接口](https://github.com/carbon-language/carbon-lang/pull/731)
> - 提案 [#1013：泛型：使用 `where` 约束设置关联的常量](https://github.com/carbon-language/carbon-lang/pull/1013)

### 通用实体

许多 Carbon 实体（不仅仅是函数）可以通过添加[已检查参数或模板参数](#checked-and-template-parameters)来成为通用实体。

#### 泛型类

可以使用可选的显式参数列表定义类。类的所有参数都必须是泛型的，因此使用 定义，可以带前缀，也可以不带前缀。例如，要定义一个可以保存任何类型的值的堆栈：`:!``template``T`

```
class Stack(T:! Type) {
  fn Push[addr me: Self*](value: T);
  fn Pop[addr me: Self*]() -> T;

  var storage: Array(T);
}

var int_stack: Stack(i32);
```

在此示例中：

- `Stack`是由类型 参数化的类型。`T`
- `T`可以在定义任何将使用正常类型的位置使用。`Stack`
- `Array(T)`实例化泛型类型，其参数设置为 。`Array``T`
- `Stack(i32)`设置为 的实例化。`Stack``T``i32`

类型参数的值是类型值的一部分，因此可以在函数调用中推导出来，如以下示例所示：

```
fn PeekTopOfStack[T:! Type](s: Stack(T)*) -> T {
  var top: T = s->Pop();
  s->Push(top);
  return top;
}

// `int_stack` has type `Stack(i32)`, so `T` is deduced to be `i32`.
PeekTopOfStack(&int_stack);
```

> 引用：
>
> - [泛型或参数化类型](generics/details.md#parameterized-types)
> - 提案 [#1146：泛型详细信息 12：参数化类型](https://github.com/carbon-language/carbon-lang/pull/1146)

#### 泛型选择类型

[选项类型](#choice-types)可以像类一样进行参数化：

```
choice Result(T:! Type, Error:! Type) {
  Success(value: T),
  Failure(error: Error)
}
```

#### 通用接口

接口始终按类型参数化，但在某些情况下，它们将具有其他参数。`Self`

```
interface AddWith(U:! Type);
```

对于给定类型，不带参数的接口只能实现一次，但类型可以具有不同的 和 实现。`AddWith(i32)``AddWith(BigInt)`

接口的参数*确定*为类型选择的实现，而[关联类型](#associated-types)*则由*类型的接口实现确定。

> 引用：
>
> - [通用或参数化接口](generics/details.md#parameterized-interfaces)
> - 提案 [#731：泛型详细信息 2：适配器、关联类型、参数化接口](https://github.com/carbon-language/carbon-lang/pull/731)

#### 泛型实现

可以通过在关键字引入符之后添加*泛型参数列表*来参数化声明，如下所示：`impl``forall [``]``impl`

```
external impl forall [T:! Printable] Vector(T) as Printable;
external impl forall [Key:! Hashable, Value:! Type]
    HashMap(Key, Value) as Has(Key);
external impl forall [T:! Ordered] T as PartiallyOrdered;
external impl forall [T:! ImplicitAs(i32)] BigInt as AddWith(T);
external impl forall [U:! Type, T:! As(U)]
    Optional(T) as As(Optional(U));
```

泛型实现可能会造成多个定义应用于给定类型和接口查询的情况。[特化](generics/details.md#lookup-resolution-and-specialization)规则选择选择哪个定义。这些规则确保：`impl`

- 实现具有[一致性](generics/terminology.md#coherence)，因此始终为给定查询选择相同的实现。
- 只要图书馆通过单独的检查，它们就会一起工作。
- 泛型函数可以假定，如果可以看到应用的 impl，即使它可以选择另一个更具体的 impl，也会成功选择某个 impl。

实现可能被标记为[`最终`](generics/details.md#final-impls)状态，以指示它们可能不是专用的，但受到[一些限制](generics/details.md#libraries-that-can-contain-final-impls)。

> 引用：
>
> - [泛型或参数化 impls](generics/details.md#parameterized-impls)
> - 提案[#624：一致性：术语，理由，考虑的替代方案](https://github.com/carbon-language/carbon-lang/pull/624)
> - 提案 [#920：通用参数化 impls（详细信息 5）](https://github.com/carbon-language/carbon-lang/pull/920)
> - 提案[#983：泛型细节7：最终imls](https://github.com/carbon-language/carbon-lang/pull/983)
> - 问题线索问题 [1192：参数化 impl 语法](https://github.com/carbon-language/carbon-lang/issues/1192)
> - 提案[#1327：泛型：`impl forall`](https://github.com/carbon-language/carbon-lang/pull/1327)

### 其他特点

碳仿制药还具有许多其他功能，包括：

- [命名约束](generics/details.md#named-constraints)可用于在组合两个具有名称冲突的接口时消除歧义。可以实现命名约束，并以其他方式用于代替接口。
- [模板约束](generics/details.md#named-constraints)是一种可以包含结构要求的命名约束。例如，模板约束可以与具有具有特定名称和签名的函数的任何类型匹配，而无需显式声明该类型实现该约束。模板约束只能用作模板参数的要求。
- [适配器类型](generics/details.md#adapting-types)是与现有类型具有相同数据表示形式的类型，因此您可以在这两种类型之间进行强制转换，但可以实现不同的接口或以不同的方式实现接口。
- 可以使用 [`where` 约束](generics/details.md#where-constraints)对接口的关联类型提出其他要求。
- [隐含约束](generics/details.md#implied-constraints)允许从函数签名中推导出和省略某些约束。
- [动态擦除的类型](generics/details.md#runtime-type-fields)可以保存具有实现接口的类型的任何值，并允许使用[动态调度](https://en.wikipedia.org/wiki/Dynamic_dispatch)调用该接口中的函数，对于某些标记为“-safe”的接口。**注意：**临时。`dyn`
- [Variadics](generics/details.md#variadic-arguments) 支持可变长度参数列表。**注意：**临时。

> 引用：
>
> - [泛型详细信息](generics/details.md)
> - 提案 [#553：泛型详述第 1 部分](https://github.com/carbon-language/carbon-lang/pull/553)
> - 提案 [#731：泛型详细信息 2：适配器、关联类型、参数化接口](https://github.com/carbon-language/carbon-lang/pull/731)

> - 提案[#818：泛型的约束（泛型细节3）](https://github.com/carbon-language/carbon-lang/pull/818)

### 泛型类型相等和声明`observe`

确定两个类型在泛型上下文中是否必须相等通常是不可判定的，如 [Swift 中所示](https://forums.swift.org/t/swift-type-checking-is-undecidable/39024)。

为了使编译速度更快，Carbon 编译器会将其搜索深度限制为 1，仅当代码中有显式声明类型相等（例如 where [``约束](generics/details.md#where-constraints)）时才将其标识为相等。在某些情况下，由于组合了这些事实，两个类型必须相等，但编译器将返回类型错误，因为它没有意识到由于搜索的限制而它们相等。[`观察`...`==`声明](generics/details.md#observe-declarations)可以添加来描述两种类型如何相等，允许更多的代码通过类型检查。

显示类型相等的声明可以增加编译器知道类型实现的接口集。了解一个类型还可能意味着它实现了另一个接口，即从[接口要求](generics/details.md#interface-requiring-other-interfaces)或[泛型实现](#generic-implementations)。一。。。声明可用于[观察类型实现接口](generics/details.md#observing-a-type-implements-an-interface)的情况。`observe``observe``is`

> 引用：
>
> - [泛型：`遵守`声明](generics/details.md#observe-declarations)
> - [泛型：观察类型实现接口](generics/details.md#observing-a-type-implements-an-interface)
> - 提案[#818：泛型的约束（泛型细节3）](https://github.com/carbon-language/carbon-lang/pull/818)
> - 提案 [#1088：通用详细信息 10：接口实现的要求](https://github.com/carbon-language/carbon-lang/pull/1088)

### 运算符过载

[表达式](#expressions)中运算符的使用将转换为对接口方法的调用。例如，如果 具有类型并且具有 类型 ，则转换为对 的调用。因此，运算符的重载是通过实现类型 的接口来完成的。为了支持将第一个操作数[隐式转换为](expressions/implicit_conversions.md)类型，将第二个参数隐式转换为类型，请在声明中将关键字添加到这两个类型中，如下所示：`x``T``y``U``x + y``x.(AddWith(U).Op)(y)``+``AddWith(U)``T``T``U``like``impl`

```
external impl like T as AddWith(like U) where .Result == V {
  // `Self` is `T` here
  fn Op[me: Self](other: U) -> V { ... }
}
```

当操作数类型和结果类型都相同时，这等效于实现接口：`Add`

```
external impl T as Add {
  fn Op[me: Self](other: Self) -> Self { ... }
}
```

对应于每个运算符的接口由下式给出：

- [算术](expressions/arithmetic.md#extensibility)：
  - `-x`:`Negate`
  - `x + y`：或`Add``AddWith(U)`
  - `x - y`：或`Sub``SubWith(U)`
  - `x * y`：或`Mul``MulWith(U)`
  - `x / y`：或`Div``DivWith(U)`
  - `x % y`：或`Mod``ModWith(U)`
- [按位和移位运算符](expressions/bitwise.md#extensibility)：
  - `^x`:`BitComplement`
  - `x & y`：或`BitAnd``BitAndWith(U)`
  - `x | y`：或`BitOr``BitOrWith(U)`
  - `x ^ y`：或`BitXor``BitXorWith(U)`
  - `x << y`：或`LeftShift``LeftShiftWith(U)`
  - `x >> y`：或`RightShift``RightShiftWith(U)`
- 比较：
  - `x == y`，通过实现 [Eq 或 `EqWith（U）```](expressions/comparison_operators.md#equality) 而重载`x != y`
  - `x < y`、 、 、 通过实现 [`Ordered` 或 `OrderedWith（U）`](expressions/comparison_operators.md#ordering) 重载`x > y``x <= y``x >= y`
- 转换：
  - `x as U`被重写为使用 [`As（U）`](expressions/as_expressions.md#extensibility) 接口
  - 隐式转换使用[`隐式As（U）`](expressions/implicit_conversions.md#extensibility)
- **待办事项：**[分配](#assignment-statements)：、、等`x = y``++x``x += y`
- **待办事项：**引用：`*p`
- **待办事项：**[移动](#move)：`~x`
- **待办事项：**索引：`a[3]`
- **待办事项：**函数调用：`f(4)`

[逻辑运算符不能重载](expressions/logical_operators.md#overloading)。

生成 [l 值的](#value-categories-and-value-phases)运算符（如取消引用和索引）具有返回值地址的接口。碳会自动取消引用指针以获取 l 值。`*p``a[3]`

可以采用多个参数的运算符（如函数调用运算符 ）具有[可变参数](generics/details.md#variadic-arguments)列表。`f(4)`

值是否支持其他操作（如复制、交换或设置为[未形成状态](#unformed-state)）是否以及如何支持其他操作，也通过实现值类型的相应接口来确定。

> 引用：
>
> - [运算符过载](generics/details.md#operator-overloading)
> - 提案[#702：比较运算符](https://github.com/carbon-language/carbon-lang/pull/702)
> - 提案 [#820：隐式转换](https://github.com/carbon-language/carbon-lang/pull/820)
> - 提案[#845：作为表达式](https://github.com/carbon-language/carbon-lang/pull/845)
> - 潜在顾客问题 [#1058：核心功能的接口应如何命名？](https://github.com/carbon-language/carbon-lang/issues/1058)
> - 提案 [#1083：算术表达式](https://github.com/carbon-language/carbon-lang/pull/1083)
> - 提案 [#1191：按位运算符](https://github.com/carbon-language/carbon-lang/pull/1191)
> - 提案 [#1178：返工操作员界面](https://github.com/carbon-language/carbon-lang/pull/1178)

#### 通用类型

在某些情况下，需要两种类型的通用类型：

- 条件[表达式（如 `c 则 t 否则 f`](expressions/if.md)）返回一个具有公用类型的值 和 。`t``f`

- 如果具有类型参数的函数有多个参数，则该函数将设置为相应参数的通用类型，如下所示：

  ```
  fn F[T:! Type](x: T, y: T);
  
  // Calls `F` with `T` set to the
  // common type of `G()` and `H()`:
  F(G(), H());
  ```

- 具有自动返回类型的函数的推断[``返回类型](#auto-return-type)是其语句的通用类型。`return`

通过实现接口来指定公共类型：`CommonTypeWith`

```
// Common type of `A` and `B` is `C`.
impl A as CommonTypeWith(B) where .Result == C { }
```

公共类型必须是两种类型都具有[隐式转换](expressions/implicit_conversions.md)的类型。

> 引用：
>
> - [`如果`表达式](expressions/if.md#finding-a-common-type)
> - 提案[#911：条件表达式](https://github.com/carbon-language/carbon-lang/pull/911)
> - 针对潜在客户的问题 [#1077：找到一种方法来允许 CommonType 的 impls 与 LHS 和 RHS 类型重叠](https://github.com/carbon-language/carbon-lang/issues/1077)

## 与 C 和 C++ 的双向互操作性

互操作性（*interop）*是调用C和从Carbon代码C++代码的能力，反之亦然。这种能力实现了两个目标：

- 允许与 C 和 C++ 共享代码和库生态系统。
- 允许从 C 和C++增量迁移到碳。

Carbon的互操作方法与[Java / Kotlin互操作](interoperability/philosophy_and_goals.md#other-interoperability-layers)最相似，其中两种语言是不同的，但共享足够的运行时模型，以便一端的数据可以从另一端使用。例如，C++和Carbon将使用相同的[内存模型](https://en.cppreference.com/w/cpp/language/memory_model)。

碳和C++之间的互操作性设计取决于：

1. 能够与各种代码（如类/结构和[模板](https://en.wikipedia.org/wiki/Template_(C%2B%2B))）进行互操作，而不仅仅是自由函数。
2. 愿意将C++习语公开到Carbon代码中，反之亦然，在必要时最大限度地提高互操作性层的性能。
3. 使用包装器和泛型编程（包括模板）来最小化或消除运行时开销。

此功能将有一些限制;只有一部分碳 API 可供C++，C++ API 的子集将可供 Carbon 使用。

- 为了实现Carbon的简化，其编程模型将排除一些很少使用和复杂的C++特征。例如，多重[继承](https://en.wikipedia.org/wiki/Multiple_inheritance)将受到限制。
- C或C++功能会损害不使用该功能的代码的性能，例如[RTTI](https://en.wikipedia.org/wiki/Run-time_type_information)和[异常](https://en.wikipedia.org/wiki/Exception_handling)，尤其需要在Carbon中进行修订。

> 引用：
>
> - [与 C/C++ 的双向互操作性](interoperability/README.md)
> - 提案[175：C++互操作性目标](https://github.com/carbon-language/carbon-lang/pull/175)

### 目标

[互操作的目标](interoperability/philosophy_and_goals.md#goals)包括：

- [支持混合碳和C++工具链](interoperability/philosophy_and_goals.md#support-mixing-carbon-and-c-toolchains)
- [与C++内存型号的兼容性](interoperability/philosophy_and_goals.md#compatibility-with-the-c-memory-model)
- [最小化桥接代码](interoperability/philosophy_and_goals.md#minimize-bridge-code)
- [C++和碳类型之间的映射并不奇怪](interoperability/philosophy_and_goals.md#unsurprising-mappings-between-c-and-carbon-types)
- [允许在 Carbon 文件中C++桥接代码](interoperability/philosophy_and_goals.md#allow-c-bridge-code-in-carbon-files)
- [C++类型的碳遗传](interoperability/philosophy_and_goals.md#carbon-inheritance-from-c-types)
- [支持使用高级C++功能](interoperability/philosophy_and_goals.md#support-use-of-advanced-c-features)
- [支持基本的 C 互操作性](interoperability/philosophy_and_goals.md#support-basic-c-interoperability)

> 引用：
>
> - [互操作性：目标](interoperability/philosophy_and_goals.md#goals)

### 非目标

[互操作的非目标](interoperability/philosophy_and_goals.md#non-goals)包括：

- [纯碳工具链与混合C++/碳工具链之间的完全奇偶校验](interoperability/philosophy_and_goals.md#full-parity-between-a-carbon-only-toolchain-and-mixing-ccarbon-toolchains)
- [从不需要网桥代码](interoperability/philosophy_and_goals.md#never-require-bridge-code)
- [将所有C++类型转换为碳类型](interoperability/philosophy_and_goals.md#convert-all-c-types-to-carbon-types)
- [支持C++异常，无需网桥代码](interoperability/philosophy_and_goals.md#support-for-c-exceptions-without-bridge-code)
- [跨语言元编程](interoperability/philosophy_and_goals.md#cross-language-metaprogramming)
- [为C++以外的语言提供等效支持](interoperability/philosophy_and_goals.md#offer-equivalent-support-for-languages-other-than-c)

> 引用：
>
> - [互操作性：非目标](interoperability/philosophy_and_goals.md#non-goals)

### 导入和`#include`

> **注意：**这是暂时的，尚未通过提案流程导入C++的设计。

C++库头文件可以使用特殊包的声明[导入](#imports)到 Carbon 中。`import``Cpp`

```
// like `#include "circle.h"` in C++
import Cpp library "circle.h";
```

这会将 中的名称添加到命名空间中。如果在作用域中定义了一些名称，则可以在Carbon的命名空间中找到这些名称。`circle.h``Cpp``circle.h``namespace shapes { ... }``Cpp.shapes`

在另一个方向上，碳包可以从C++文件中导出要d的头文件。`#include`

```
// like `import Geometry` in Carbon
#include "geometry.carbon.h"
```

通常，Carbon实体可以从C++使用，C++实体可以从Carbon使用。这包括类型、函数和常量。某些实体（如 Carbon 接口）将无法直接翻译。

定义常量的 C 和 C++ 宏将作为常量导入。否则，C 和 C++ 宏在 Carbon 中将不可用。C 和 C++ s 将被转换为类型常量，就像使用 [`let`](#constant-let-declarations) 声明一样。`typedef`

满足某些限制的碳函数和类型也可以注释为导出到C，例如C++的[`extern “C”`](https://en.wikipedia.org/wiki/Compatibility_of_C_and_C%2B%2B#Linking_C_and_C++_code)标记。

### ABI 和动态链接

> **注意：**这反映了目标和计划。尚未通过提案流程，尚未对实施进行具体设计。

Carbon本身对于整个语言来说不会有一个稳定的ABI，大多数语言功能都是围绕没有任何ABI稳定性而设计的。相反，我们希望添加专用的语言功能，这些功能专门设计用于在Carbon程序的两个独立部分之间提供ABI稳定的边界。这些 ABI 弹性语言功能和 API 边界将是选择性加入和显式的。它们还可能具有功能限制，以使其易于实施，并具有很强的ABI弹性。

当与已编译C++对象代码或共享库进行互操作时，C++互操作的功能可能明显低于其他功能。这是一个可供我们探索的开放领域，但我们预计需要重新编译C++代码，以便在与Carbon互操作时获得完整的人体工程学和性能优势。例如，重新编译让我们确保Carbon和C++可以对关键词汇表类型使用相同的表示形式。

但是，我们希望在与已编译的 C 对象代码或共享库进行互操作时完全支持 C ABI。我们希望Carbon的桥接代码功能能够涵盖与C++的[`extern“C”`](https://en.wikipedia.org/wiki/Compatibility_of_C_and_C%2B%2B#Linking_C_and_C++_code)标记类似的用例，以便在这里提供完整的双向支持。当然，跨此互操作边界可用的功能将仅限于 C ABI 中可表达的功能，并且类型可能需要显式标记才能保证 ABI 兼容性。

> 引用：
>
> - [目标：稳定的语言和库 ABI 非目标](https://github.com/carbon-language/carbon-lang/blob/trunk/docs/project/goals.md#stable-language-and-library-abi)
> - [#175：C++互操作性目标：支持混合碳和C++工具链]()

### 运算符过载

> **注意：**这是暂时的，尚未通过提案流程进行设计。

Carbon支持[运算符重载](#operator-overloading)，但通过[实现接口](#interfaces-and-implementations)而不是像C++中那样定义方法或非成员函数来完成。

使用接口实现运算符重载的 Carbon 类型应在C++中获得相应的运算符重载。因此，在 Carbon 中为某个类型实现可以有效地实现该类型的C++。这也适用于另一个方向，因此实现运算符重载C++类型会自动考虑实现相应的 Carbon 接口。因此，在C++中实现一个类型也实现了Carbon中的接口。但是，隐式转换或重载选择可能存在边缘情况，这些情况不能完全映射到Carbon中。`ModWith(U)``operator%``operator%``ModWith(U)`

在某些情况下，操作在两种语言中的编写方式可能不同。在这些情况下，它们根据哪个操作具有最相似的语义而不是使用相同的符号进行匹配。例如，Carbon 中的操作和接口对应于C++中的操作和函数。类似地，Carbon 接口对应于 C++ 中的隐式转换，可以通过多种不同的方式编写。其他[C++自定义点](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html)（如将逐个与 Carbon 接口相对应）。`^x``BitComplement``~x``operator~``ImplicitAs(U)``swap`

某些运算符仅在C++中存在或可重写，例如逻辑运算符或逗号运算符。在不太可能的情况下，需要为Carbon类型覆盖这些运算符，这可以通过非成员C++函数来完成。

没有C++等效项的碳接口，例如[`CommonTypeWith（U），`](#common-type)可以在Carbon代码中外部实现C++类型。为了满足孤立规则 （[1](generics/details.md#impl-lookup)， [2](generics/details.md#orphan-rule)），每个C++库将具有一个相应的 Carbon 包装器库，如果 Carbon 包装器存在，则必须导入该库，而不是C++库。**待办事项：**也许它将自动导入，因此可以在不需要更改导入器的情况下添加包装器？

### 模板

> **注意：**这是暂时的，尚未通过提案流程进行设计。

Carbon 支持[已检查的泛型和模板泛型](#checked-and-template-parameters)。这为C++模板代码提供了迁移路径：

- C++模板 -> Carbon 模板：这涉及将代码从 C++ 迁移到 Carbon。如果该迁移是忠实的，则更改对呼叫者应该是透明的。
- ->具有约束的碳模板：约束可以一次添加一个。只要代码继续编译，添加约束永远不会改变代码的含义。编译错误将指向需要实现缺少接口的类型。该接口的临时模板实现可以在转换期间充当桥梁。
- -> Carbon 选中的泛型：添加完所有约束后，所有调用方都工作，模板参数可以切换到选中的泛型。

Carbon还将以多种方式提供与C++模板的直接互操作：

- 能够调用C++模板并使用 Carbon 中的C++模板化类型。
- 能够使用 Carbon 类型实例化C++模板。
- 能够使用C++类型实例化碳泛型。

我们希望这些领域的最佳互操作基于Carbon提供的C++工具链。但是，即使将Carbon生成的C++标头用于互操作，我们也将尽可能包括使用来自C++的Carbon通用代码的功能，就好像它是C++模板一样。

### 标准类型

> **注意：**这是暂时的，尚未通过提案流程进行设计。

碳整数类型（如 和 ）被视为等于 C++ 中相应的固定宽度整数类型，如 和 ，由 或 提供。基本的 C 和 C++整数类型，如 、 、，并且在给定声明的命名空间内的 Carbon 中可用，其名称如 、 、 和 。如果C++认为C++类型不同，则认为它们不同，因此C++重载的解析方式相同。[整数类型之间隐式转换的碳约定](expressions/implicit_conversions.md#data-types)在这里适用，只要转换可以保留所有输入的数值，就允许它们。`i32``u64``int32_t``uint64_t``<stdint.h>``<cstdint>``int``char``unsigned long``Cpp``import Cpp;``Cpp.int``Cpp.char``Cpp.unsigned_long`

其他 C 和 C++ 类型等于碳类型，如下所示：

| C 或 C++ | 碳             |
| :------- | :------------- |
| `bool`   | `bool`         |
| `float`  | `f32`          |
| `double` | `f64`          |
| `T*`     | `Optional(T*)` |
| `T[4]`   | `[T; 4]`       |

此外，C++引用类型（如 Carbon）将被转换为 Carbon，这是 Carbon 的非空指针类型。`T&``T*`

Carbon将致力于为常见数据结构提供惯用词汇*视图*类型，例如和，在C++和Carbon等价物之间透明映射。这将包括数据布局，以便即使指向这些类型的指针也能无缝转换，具体取决于这些类型的合适C++ ABI，可能是通过使用自定义的ABI重新编译C++代码。我们还将探讨如何将覆盖范围扩展到其他库中的类似视图类型。`std::string_view``std::span`

但是，Carbon的容器将不同于C++标准库容器，以最大限度地提高我们提高性能的能力，并在设计和实现中利用语言功能，如检查泛型。

在可能的情况下，我们还将尝试为相关C++容器类型提供Carbon标准库容器*接口*的实现，以便它们可以直接与通用Carbon代码一起使用。这应该允许Carbon中的通用代码与Carbon和C++容器无缝协作，而不会降低性能或限制Carbon容器的实现。另一方面，碳容器将满足C++容器要求，因此模板化的C++代码也可以直接在碳容器上运行。

### 遗产

[Carbon具有单一继承](#inheritance)，允许迁移使用继承的C++类。数据表示将是一致的，因此Carbon类可以从C++类继承，反之亦然，即使使用虚拟方法也是如此。

C++[多重继承](https://en.wikipedia.org/wiki/Multiple_inheritance)，[CRTP](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern) 将使用 Carbon 特征的组合进行迁移。碳混合蛋白支持实现重用，碳接口允许一个类型实现多个 API。但是，跨C++ <->碳边界的多重继承的互操作可用程度可能存在限制。

碳 dyn 安全接口可以作为[抽象基类](https://en.wikipedia.org/wiki/Class_(computer_programming)#Abstract_and_concrete)导出到C++。也可以使用代理对象实现C++抽象基类并持有指向实现相应接口的类型的指针来执行反向操作。

> 引用：
>
> - 提案 [#561：基本类：用例、结构文字、结构类型和未来工作](https://github.com/carbon-language/carbon-lang/pull/561)
> - 提案[#777：继承](https://github.com/carbon-language/carbon-lang/pull/777)

### 枚举

> **待办事项**

## 未完成的故事

> **注意：**本节中的所有内容都是临时的和前瞻性的。

### 安全

Carbon的前提是，C++用户不能为了获得安全而放弃性能。即使一些孤立的用户能够做出这种权衡，他们也会与对性能敏感的用户共享代码。任何安全途径都必须保持当今C++的性能。这排除了垃圾回收和许多其他选项。在不放弃性能的情况下实现安全性的唯一很好理解的机制是编译时安全性。如何实现这一目标的主要例子是 Rust。

Rust的方法和Carbon的方法之间的区别在于，Rust从安全开始，Carbons从迁移开始。Rust 支持与 C 的互操作，并且正在进行改进C++互操作故事和开发迁移工具的工作。但是，两种语言之间的编程模型存在很大差距，通常需要对体系结构进行修订。因此，到目前为止，Rust 社区的常见模式是“在 Rust 中重写它”（[1](https://deprogrammaticaipsum.com/the-great-rewriting-in-rust/)，[2](https://unhandledexpression.com/rust/2017/07/10/why-you-should-actually-rewrite-it-in-rust.html)，[3](https://transitiontech.ca/random/RIIR)）。Carbon的方法是专注于从C++迁移，包括无缝互操作，然后逐步提高安全性。

支持其安全策略对Carbon设计的第一个影响是这种编译时安全级别的必要构建块。我们研究了像 Rust 和 Swift 这样的现有语言，以了解它们最终需要哪些基本功能。突出的两个组件是：

- 包含更多语义信息的扩展类型系统。
- 更普遍地使用类型系统抽象（通常是泛型）。

为了迁移C++代码，我们还需要能够添加功能和迁移代码，以便随着时间的推移逐步使用这些新功能。这需要在第一天就设计出具有进化的语言。这会影响广泛的功能：

- 在最低级别，简单且可扩展的语法和语法。
- 用于添加和删除 API 的工具和支持。
- 可扩展的迁移策略，包括工具支持。

Rust 显示了类型系统中扩展语义信息的价值，例如精确的生存期。这在C++中很难做到，因为它有太多的引用和指针，这会增加类型系统的复杂性。Carbon试图将C++的类型变化压缩为值和[指针](#pointer-types)。

Rust 还显示了按生存期参数化的函数的值。由于生存期仅用于建立代码的安全属性，因此没有理由为这些参数支付单态化的成本。因此，我们需要一个[泛型系统](#generics)，可以在代码实例化之前对其进行推理，这与C++模板不同。

总之，碳如何从C++中分化有两种模式：

- 简化和移除事物，为新的安全功能创造空间。这微不足道地需要破坏向后兼容性。
- 重新设计基础以建模和实施安全性。这在C++方面具有复杂性和困难性，而无需首先简化语言。

这导致了Carbon的增量安全路径：

- 保持性能、现有代码库和开发人员。
- 通过可扩展的工具辅助迁移从C++采用 Carbon。
- 从第一天开始，就解决最初的、简单的安全改进问题。
- 在未来十年内，将 Carbon 代码转移到内存安全的增量路径上。

> 参考：[安全策略]()

### 生存期和移动语义

> **待办事项：**

### 元编程

> **待办事项：**参考文献需要发展。需要详细的设计和内联提供的高级摘要。

Carbon 提供了类似于常规 Carbon 代码的元编程工具。这些是结构化的，并且不提供对源文本的任意包含或预处理，例如C和C++。

> 参考文献：[元编程](metaprogramming.md)

### 模式匹配作为函数重载分辨率

> **待办事项：**参考文献需要发展。需要详细的设计和内联提供的高级摘要。

> 参考：[模式匹配](pattern_matching.md)

### 错误处理

目前，Carbon没有专门用于错误处理的语言功能，但我们会考虑在未来添加一些功能。此时，错误使用[选择类型](#choice-types)（如 和 ）表示。`Result``Optional`

这类似于 Rust 的故事，Rust 开始使用 ，然后为了方便起见添加了 [`？` 运算符](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-question-mark-operator)，现在正在考虑 （[1](https://yaah.dev/try-blocks)， [2](https://doc.rust-lang.org/beta/unstable-book/language-features/try-blocks.html)） 添加更多。`Result`

### 执行抽象

Carbon提供了程序执行的一些高阶抽象，以及这些抽象的关键基础。

#### 抽象机器和执行模型

> **待办事项：**

#### Lambdas

> **待办事项：**

#### 共同例程

> **待办事项：**

#### 并发

> **待办事项：**