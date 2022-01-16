---
 title: Rust 2021 版入门
 date: 2022-01-16
 tag: rust
---
# rust

## Defining Generic Functions and Types （定义泛型函数和类型）

*  How to write a single function definition, whose invocations can efficiently handle different data types （如何编写单个函数定义，其调用可以有效地处理不同的数据类型）
* How to use type inference to avoid the need to specify the types used by generic functions（如何使用类型推断来避免需要指定泛型函数使用的类型）
* How to write a single struct, tuple-struct, or enum type, whose instances can efficiently contain different data types（如何编写单个结构、元组结构或枚举类型，其实例可以有效地包含不同的数据类型）
* How to use two important standard generic enums, Option and Result, to represent optional data or fallible functions results（如何使用两个重要的标准通用枚举 Option 和 Result 来表示可选数据或易出错的函数结果”）
* How some standard functions ease the handling of Option and Result（一些标准函数如何简化选项和结果的处理）

### Need of Generic Functions  （需要泛型函数）


