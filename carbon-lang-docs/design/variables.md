# 变量

## 目录

- [概述](#概述)
- [笔记](#笔记)
  - [全局变量](#全局变量)
- [考虑的替代方案](#考虑的替代方案)
- [引用](#引用)

## 概述

Carbon的局部变量语法是：

- `var` `变量名` `类型或者auto` : `值`

大括号引入了嵌套作用域，并且可以包含与函数参数类似的局部变量声明。

例如：

```
fn Foo() {
  var x: i32 = 42;
}
```

这里引入了一个局部变量x，其作用域在大括号之间的范围。它是int型变量并且值为42。现在我们介绍的关于变量（和函数的声明）仅仅是冰山一角，但是这提供了一种基本的声明方式。

如果用于代替类型，则[使用类型推断](type_inference.md)来自动确定变量的类型：`auto`

虽然可以有全局常量，但没有全局变量。

## 笔记

> TODO：常量语法是一个持续的讨论。

### 全局变量

我们正在探索几种不同的想法，以设计不易出错的模式，以取代程序员仍然对全局变量的重要用例。我们可能无法完全解决它们，至少对于迁移的代码，并被迫添加一些有限形式的全局变量。我们还可能发现，它们的便利性超过了所提供的任何改进。

## 考虑的替代方案

- [无 `var` 引入器关键字]()
- [`var` 语句引入器的名称]()
- [类型和标识符之间的冒号]()
- [类型省略]()
- [类型排序]()
- [省略类型而不是使用`自动`]()

## 引用

- 提案 [#339：`var` 语句](https://github.com/carbon-language/carbon-lang/pull/339)
- 提案[#618：`var`排序](https://github.com/carbon-language/carbon-lang/pull/618)
- 提案 [#851：vars 的自动关键字](https://github.com/carbon-language/carbon-lang/pull/851)