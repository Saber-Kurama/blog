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

### Inferring the Parametric Types （推断参数类型）

```rust
// Library code
fn f<T>(ch: char, num1: T, num2: T) -> T {
    if ch == 'a' { num1 }
    else { num2 }
}
// Application code 
// 类型推断
let a: i16 = f('a', 37, 41);
let b: f64 = f('b', 37.2, 41.1);
print!("{} {}", a, b);
```

多个泛型参数

```rust
fn f<Param1, Param2>(_a: Param1, _b: Param2) {}
f('a', true);
f(12.56, "Hello");
f((3, 'a'), [5, 6, 7]);
```

### Defining and Using Generic Structs （定义和使用泛型结构）

参数类型对于声明泛型结构和泛型元组结构也很有用：

``` rust
#[allow(dead_code)]
struct S<T1, T2> {
    c: char,
    n1: T1,
    n2: T1,
    n3: T2,
}
let _s = S { c: 'a', n1: 34, n2: 782, n3: 0.02 };
struct SE<T1, T2> (char, T1, T1, T2);
let _se = SE ('a', 34, 782, 0.02);
```
同样对于结构，类型参数的具体化可以明确

```rust
#[allow(dead_code)]
struct S<T1, T2> {
    c: char,
    n1: T1,
    n2: T1,
    n3: T2,
}
let _s = S::<u16, f32> { c: 'a', n1: 34, n2: 782, n3: 0.02 };
struct SE<T1, T2> (char, T1, T1, T2);
let _se = SE::<u16, f32> ('a', 34, 782, 0.02);
```

### Genericity Mechanics  (泛型原理？)

为了更好地理解泛型是如何工作的，你应该扮演编译器的角色并遵循编译过程。事实上，从概念上讲，通用代码编译发生在几个阶段
让我们遵循编译的概念机制，应用于以下代码，其中包括 main 函数

```rust
fn swap<T1, T2>(a: T1, b: T2) -> (T2, T1) { (b, a) }
fn main() {
    let x = swap(3i16, 4u16);
    let y = swap(5f32, true);
    print!("{:?} {:?}", x, y);
}
```

在第一阶段，扫描源代码，每次编译器找到一个泛型函数声明（在示例中是交换函数的声明）时，它都会在其数据结构中加载该函数的内部表示，总共它的通用性，只检查通用代码中没有语法错误。

在第二阶段，再次扫描源代码，每次编译器遇到泛型函数的调用时，它都会在其数据结构中加载这种用法与泛型声明的相应内部表示之间的关联，当然之后已检查该等函件是否有效。

因此，在我们示例的前两个阶段之后，编译器有一个通用交换函数和一个具体的主函数，最后一个函数包含对通用交换函数的两个引用

在第三阶段，扫描所有泛型函数的调用（在示例中，两次调用 swap）。对于每个这样的用法，以及对应定义的每个泛型参数，都确定了一个具体类型。这种具体类型可以显式指定，或者（如在示例中）可以从用作函数参数的表达式的类型中推断出来。示例中，第一次调用swap，参数T1关联到i16类型，参数T2关联到u16类型；在第二次调用 swap 时，参数 T1 关联到 f32 类型，参数 T2 关联到 bool 类型。
在确定了要替换泛型参数的具体类型之后，将生成泛型函数的具体版本。在这样的具体版本中，每个泛型参数都被为特定函数调用确定的具体类型所替换，并且泛型函数的调用被刚刚生成的具体函数的调用所替换。
例如，生成的内部表示对应于以下 Rust 代码：

```rust
fn swap_i16_u16(a: i16, b: u16) -> (u16, i16) { (b, a) }
fn swap_f32_bool(a: f32, b: bool) -> (bool, f32) { (b, a) }
fn main() {
	let x = swap_i16_u16(3i16, 4u16);
    let y = swap_f32_bool(5f32, true);
    print!("{:?} {:?}", x, y);
}
```

如您所见，没有更多的通用定义或通用函数调用。通用函数定义已转换为两个具体函数定义，两个函数调用现在各自调用不同的具体函数。
第四阶段包括编译此代码。
请注意，需要生成两个不同的具体函数，因为对通用交换函数的两次调用指定了不同的类型。
但是这段代码:

```rust
fn swap<T1, T2>(a: T1, b: T2) -> (T2, T1) { (b, a) }
let x = swap('A', 4.5);
let y = swap('g', -6.);
print!("{:?} {:?}", x, y);
```

在内部翻译成这个代码：

```rust
fn swap_char_f64(a: char, b: f64) -> (f64, char) { (b, a) }
let x = swap_char_f64('A', 4.5);
let y = swap_char_f64('g', -6.);
print!("{:?} {:?}", x, y);
```
即使对通用函数交换有多次调用，也只会生成一个具体版本，因为所有调用都需要相同的参数类型

一般来说，当多个调用指定完全相同的类型参数时，它总是应用只生成一个具体版本的泛型函数声明的优化。
编译器可以在单个程序中生成对应于单个函数的多个具体版本的机器代码这一事实会产生以下后果：
*  就编译非泛型代码而言，这种多阶段编译稍微慢一些。
* 生成的代码针对每个特定调用进行了高度优化，因为它完全使用调用者使用的类型，而不需要转换或决策。因此，每次调用的运行时性能都得到了优化。
* 如果对一个通用函数执行许多具有不同数据类型的调用，则会生成大量机器代码，并且由于代码局部性差，这可能会影响性能。为了减少这种称为代码膨胀的现象，最好减少用于调用泛型函数的不同类型的数量。

本节中关于泛型函数的所有内容也适用于泛型结构和元组结构

### Generic Arrays and Vectors (泛型数组和向量)

关于数组和向量，还没有消息。我们从第 5 章介绍它们的地方看到，它们是泛型类型。
实际上，虽然数组是 Rust 语言的一部分，但向量是在 Rust 标准库中定义的结构。

### Generic Enums （泛型枚举）
在 Rust 中，甚至枚举也可以是泛型的。

``` rust
enum Result1<SuccessCode, FailureCode> {
    Success(SuccessCode),
    Failure(FailureCode, char),
    Uncertainty,
}
let mut _res = Result1::Success::<u32, u16>(12u32);
_res = Result1::Uncertainty;
_res = Result1::Failure(0u16, 'd');
```
前面的程序是有效的，但是下面的程序会在最后一行导致编译：

```rust
enum Result1<SuccessCode, FailureCode> {
    Success(SuccessCode),
    Failure(FailureCode, char),
    Uncertainty,
}
let mut _res = Result1::Success::<u32, u16>(12u32);
_res = Result1::Uncertainty;
_res = Result1::Failure(0u32, 'd');
```

这里，Failure的第一个参数是u32类型，而应该是u16类型，根据_res的初始化，前两行。
泛型枚举在 Rust 标准库中被大量使用。

