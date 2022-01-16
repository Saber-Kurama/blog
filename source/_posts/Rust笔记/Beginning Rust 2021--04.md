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

泛型函数可以解决函数多态的问题，参数类型不同，方法名称一致

``` rust
square_root(x: f32) -> f32; // 这样的方法 如果没有泛型 必须将参数 转换
square_root(45.2f64 as f3)
```

### Defining and Using Generic Functions （定义和使用泛型函数）

``` rust
// Library code
fn f<T>(ch: char, num1: T, num2: T) -> T {
    if ch == 'a' { num1 }
    else { num2 }
}
// Application code
let a: i16 = f::<i16>('a', 37, 41);
let b: f64 = f::<f64>('b', 37.2, 41.1);
print!("{} {}", a, b); // 37  41.1
```

在函数定义中，就在函数名称之后，有一个用尖括号括起来的 T 字。该符号是函数声明的类型参数。
这意味着声明的不是具体函数，而是泛型函数，由 T 类型参数参数化。仅当仍在编译时为此类 T 参数指定具体类型时，该函数才会成为具体函数。
T 参数仅在函数定义的范围内定义。事实上，它被使用了 3 次，不仅在函数的签名中。它也可以用在函数体中，但不能用在其他地方。

