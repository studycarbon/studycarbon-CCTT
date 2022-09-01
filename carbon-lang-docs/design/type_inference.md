# 类型推断

## 目录

- [概述](#overview)
- 开放性问题
  - [从文本推断变量类型](#inferring-a-variable-type-from-literals)
- [考虑的替代方案](#alternatives-considered)
- [引用](#references)

## 概述

使用关键字时，[类型推断](https://en.wikipedia.org/wiki/Type_inference)在 Carbon 中发生。这可能发生在[变量声明](variables.md)或[函数声明中](functions.md)。`auto`

目前，类型推断非常简单：给定生成用于类型推断的值的表达式，推断的类型是该表达式的精确类型。例如，in 的推断类型是 。`auto``fn Foo(x: i64) -> auto { return x; }``i64`

[函数返回类型](functions.md)和[声明的变量类型](variables.md)目前支持类型推断。

## 开放性问题

### 从文本推断变量类型

对当前使用右侧的类型会产生常量值，而大多数语言会建议使用可变整数类型，例如 。碳也可能使它成为一个错误。尽管类型推断目前仅针对变量和函数返回类型的地址，但通常将其视为类型推断的一部分，因为它还会影响泛型、模板、lambda 和返回类型。`var y: auto = 1``IntLiteral(1)``i64``auto`

## 考虑的替代方案

- [使用 `_` 而不是`自动`]()

## 引用

- 提案 [#851：vars 的自动关键字](https://github.com/carbon-language/carbon-lang/pull/851)