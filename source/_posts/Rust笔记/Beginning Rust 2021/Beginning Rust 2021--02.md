---
 title: Rust 2021 版入门
 date: 2022-01-11
 tag: rust
---
# Rust
## Using Data Sequences 使用序列数据
* How to define sequences of objects of the same type, having fixed length (arrays) or variable length (vectors) （固定长度的数组和可变长度的vector）
*  How to specify the initial contents of arrays or vectors, by listing the items or by specifying one item and its repeat count （如何通过列出项目或指定一个项目及其重复次数来指定数组或向量的初始内容）
* How to read or write the value of single items of arrays or vectors（如何读取或写入数组或向量单项的值）
* How to add items to a vector or to remove items from a vector（如何将项目添加到向量或从向量中删除项目）
* How to create arrays with several dimensions （如何创建具有多个维度的数组）
* How to create empty arrays or vectors （如何创建空数组或向量）
* How to print or copy whole arrays or vectors（如何打印或复制整个数组或向量）
*  How to specify whether the compiler should deny, allow with a warning, or silently allow possible programming errors（如何指定编译器是否应该拒绝、允许并发出警告，或者静默允许可能的编程错误）
* What is a panic, and why it may be generated when accessing an array or a vector （什么是恐慌，为什么在访问数组或向量时可能会产生恐慌）

### Array 

和 c 语言一样 数组的索引从0开始
可以创建任意类型数组，只要数组的类型是一致的
``` rust
let x = ["a"]; // array of strings
let _y = x[1];
```
这个例子会产生运行时错误

### Rust Attributes Rust 属性

Rust 编译器在 Rust 语言的分析严格性方面有些灵活。对于某些可能的错误情况，例如在最后一次赋值后未使用变量。
编译器允许开发人员指定三种可能的编译行为：
* Deny: The compiler emits an error message, and it does not generate the executable code.（拒绝：编译器发出错误消息，并且不生成可执行代码）
* Warn: The compiler emits a warning message, but it does generate the executable code.（警告：编译器发出警告消息，但它确实生成了可执行代码。）
* Allow: The compiler does not emit any message, and it does generate the executable code.（允许：编译器不会发出任何消息，它会生成可执行代码。）

例子：
``` rust 
#[deny(unused_variables)]
let x = 1;
#[warn(unused_variables)]
let y = 2;
#[allow(unused_variables)]
let z = 3;
```

(这里很像typescript中ts的注释， 或者是eslint)
括号中的标识符。标识符必须是已知的编译器选项


``` rust
“#[allow(unused_variables)]
fn main() {
    let x = 1;
    let y = 2;
}
```
整个程序都忽略

### Panicking 恐慌
 数组访问越界就会退出进程
 在 Rust 行话中，这样的中止操作被称为`panic`，而中止当前线程的动作被称为`panicing`

 ``` rust
 let x = ["a"];
let _y = x[1];
print!("End");
 ```
 编译的时候会报错，因为在编译阶段已经预料到会恐慌

 ``` rust
 let x = ["a"];
#[allow(unconditional_panic)]
let _y = x[1];
print!("End");
 ```
 编译通过但是运行恐慌

 ### Mutable Arrays 可变数组

 数组的修改只能在可变数组上

 不允许添加数组和删除数组一项，因为数组的长度是固定的

下面是可以的
 ``` rust
let mut x = ["a", "b", "c"];
print!("{}{}{}. ", x[0], x[1], x[2]);
x = ["X", "Y", "Z"];
print!("{}{}{}. ", x[0], x[1], x[2]);
let  y = ["1", "2", "3"];
x = y;
print!("{}{}{}.", x[0], x[1], x[2]);

```
但是 下面就会报 类型不匹配
``` rust
let mut x = ["a", "b", "c"];
x = ["X", "Y"];
x = [15, 16, 17];
```

### Arrays of Explicitly Specified Size  明确指定大小的数组

重复项创建数组
``` rust
let mut x = [4.; 5000];
x[2000] = 3.14;
print!("{}, {}", x[1000], x[2000]);
```
可以用` for ... in ...` 来 扫描数组
``` rust
let mut fib = [1; 12];
for i in 2..fib.len() {
    fib[i] = fib[i - 2] + fib[i - 1];
}
for i in 0..fib.len() {
    print!("{}, ", fib[i]);
}
```
### Multidimensional Arrays 多维数组

``` rust
let mut x = [[[23; 4]; 8]; 15];
x[14][7][3] = 56;
print!("{}, {}", x[0][0][0], x[14][7][3])
```
多维数组只不过是数组的数组

数组的一个很大限制是它们的大小必须在编译时定义, 所以下面语句是不可以
```rust
let length = 6;
let arr = [0; length];
```

### Vectors
 使用 vec宏 创建
``` rust
let x = vec!["This", "is"];
```
vector 是修改长度

``` rust
let mut x = vec!["This", "is"]; print!("{}", x.len());
x.push("a"); print!(" {}", x.len());
x.push("sentence"); print!(" {}", x.len());
x[0] = "That";
for i in 0..x.len() { print!(" {}", x[i]); }
```
下面的例子在vector中也是可以的
```rust
let length = 5000;
let mut y = vec![4.; length];
y[6] = 3.14;
y.push(4.89);
print!("{}, {}, {}", y[6], y[4999], y[5000])
```
与数组不同，可以创建在运行时定义长度的向量，并且您可以在初始化后在运行时更改它们的长度
Rust 数组等价于 C++ std::array 对象，而 Rust 向量等价于 C++ std::vector 对象

### Other Operations on Vectors （Vector的其他操作）
一些操作
* insert
* remove
* push
* pop
* ...

### Empty Arrays and Vectors
怎么声明一个空数组和vector呢
```rust
let _a = ["", 0];
let _b = vec![false, 0];
```

### Debug Print 打印调试
打印数组和vector
``` rust
print!("{:?} {:?}", [1, 2, 3], vec![4, 5]);
```
`{:?}` 

### Copying Arrays and Vectors

数组的复制
``` rust
let mut a1 = [4, 56, -2];
let a2 = [7, 81, 12500];
println!("{:?} {:?}", a1, a2);
a1 = a2;
println!("{:?} {:?}", a1, a2);
a1[1] = 10;
println!("{:?} {:?}", a1, a2);
```
vector 复制
与数组不同的是，当一个向量被分配给另一个向量时，原来的向量就不再存在了

## Using Primitive Types 使用原始类型
* How to write numeric literals in hexadecimal, octal, or binary notation（如何以十六进制、八进制或二进制表示法编写数字文字）
* How to use the underscore character to make numeric literals easier to read （如何使用下划线字符使数字文字更易于阅读）
* How to use the exponential notation to write huge or tiny numbers in a compact form（如何使用指数符号以紧凑的形式写出巨大或微小的数字）
* Which are the twelve primitive integer numeric types, and the two primitive floating-point numeric types; what are their ranges; and when it is better to use each of them（分别是 12 种原始整数数值类型和两种原始浮点数值类型；它们的范围是多少；以及何时更好地使用它们中的每一个）
* How to specify numeric literals of concrete types or of unconstrained types（如何指定具体类型或无约束类型的数字文字）
* How to convert a numeric value to another numeric type（如何将数值转换为另一种数值类型）
* The other primitive types: Booleans, characters, and empty tuples （其他原始类型：布尔值、字符和空元组）
* How type inference works （类型推断的工作原理）
* How to express the types of arrays and vectors（如何表达数组和向量的类型）
* How to assign a name to a compile-time constant（如何为编译时常量分配名称）
* How to use the compiler to discover the type of an expression （如何使用编译器发现表达式的类型）

### Nondecimal Numeric Bases（非十进制数字基数）

``` Rust
	let hexadecimal = 0x10;
    let octal = 0o10;
    let binary = 0b10;
    let mut n = 10;
    print!("{} ", n);
    n = hexadecimal;
    print!("{} ", n);
    n = octal;
    print!("{} ", n);
    n = binary;
    print!("{}", n);
```
浮点数只能用十进制表示.请注意，此类表示仅存在于源代码中，因为 Rust 编译器生成的机器代码始终使用二进制表示法，无论是整数数还是浮点数
用作十六进制数字的字母可以是大写或小写。例如，数字 0xAEf5b 等于数字 0xaeF5B。但是，用于表示数字基数的字母必须是小写的。因此，表达式 0X4、0O4 和 0B4 是非法的

### The Underscore in Numeric Literals (数字文字中的下划线)

```rust
let hexadecimal = 0x_00FF_F7A3;
let octal = 0o_777_205_162;
let decimal = 1_234_567;
let binary = 0b_0110_1001_1111_0001;

```
### The Exponential Notation (指数符号)
浮点数可以达到巨大的值，无论是正数还是负数，比如十亿十亿十亿，也可以是非常小的值，即非常接近零的值，比如十亿分之一的十亿分之一。十亿分之一。如果我们使用目前使用的符号来编写如此大或小的数字的文字，我们将不得不写许多零，并且生成的数字将难以阅读，即使使用下划线也是如此。
但你也可以用另一种方式编写浮点文字数字：

``` rust
let one_thousand = 1e3;
let thirteen_billions_and_half = 13.5e9;
let twelve_millionths = 12e-6; 
```
### The Various Kinds of Signed Integer Numbers (各种有符号整数)
rust 有十二种不同的整数数字类型和两种浮点数字类型
`i8`, `i16`, `i32`, `i64`, `i128`

### Unsigned Integer Number Types (无符号整数类型)
`u8`, `u16`,`u32`,`u64`,`u128`

### Target-Dependent Integer-Number Types (目标相关整数类型)
数组和vector的索引的是 无符号类型 `usize`
`isize`

### Type Inference (类型推断)

``` rust
let i = 0; // 先推断成 usize
let _j: u16 =  i; // 因为_j 是 u16，所以 确定了i类型为 u16
let _k: i16 = i; // 错误
let _k1 = i；// 这个是正确
```
### The Type Inference Algorithm （类型推断算法）

We saw that the compiler always tries to determine a concrete type for each variable and for each expression. For what we saw so far, the algorithm used is the following one:
我们看到编译器总是试图为每个变量和每个表达式确定一个具体的类型。就我们目前看到的情况而言，使用的算法如下：

If a type is explicitly specified, the type of the variable must be the specified one.

如果明确指定了类型，则变量的类型必须是指定的类型。

If the type of a variable or an expression hasn’t yet been determined at all, and such variable or expression is used in an expression or in a declaration that can be valid only with a specific type, then that type is determined in this way for such variable or expression. Such determination may be of a constrained kind, or it may be of an unconstrained kind. A constrained type is a specific type, like i8 or u64, while an unconstrained type is a category of types, like {integer}. The word constrained is used to mean that the compiler finds that the Rust syntax allows only one specific type in that context. The word unconstrained means that the compiler finds that a family of types, like all the positive integers or all the integers, is allowed in that context.

如果一个变量或表达式的类型还没有被确定，并且这样的变量或表达式被用在一个表达式或声明中，只能对特定类型有效，那么该类型在此确定这种变量或表达式的方式。这种确定可以是受约束的类型，也可以是不受约束的类型。受约束类型是特定类型，例如 i8 或 u64，而不受约束类型是类型的类别，例如 {integer}。 constraint 一词用于表示编译器发现 Rust 语法在该上下文中只允许一种特定类型。无约束一词意味着编译器发现在该上下文中允许使用一系列类型，例如所有正整数或所有整数。

If, at the end of the parsing, the compiler has determined only that a variable is of an unconstrained integer numeric type, that type is defined to be i32. Instead, if the type is completely undetermined, a compilation error is generated.

如果在解析结束时，编译器仅确定变量是无约束的整数数字类型，则该类型被定义为 i32。相反，如果类型完全不确定，就会产生编译错误。

### Floating-Point Numeric Types 浮点数值类型

Rust 中只有两种浮点类型：`f32` 和`f64`

### Explicit Conversions  显式转换

``` rust
let a: i16 = 12;
let b: u32 = 4;
let c: f32 = 3.7;
print!("{}", a as i8 + b as i8 + c as i8);
```

下面这个会报错
```rust
let a = 500 as i8;
let b = 100_000 as u16;
let c = 10_000_000_000 as u32;
print!("{} {} {}", a, b, c);
```
第一个有消息：i8 的字面量超出范围。发生这种情况是因为值 500 对于 i8 类型来说太大了。
第二行和第三行发出类似的消息
但是，如果您在程序的开头插入以下行：
``` rust
#[allow(overflowing_literals)]

```

### Type Suffixes of Numeric Literals  数字文字的类型后缀

``` rust
let _a: i16 = -150;
let _b = -150 as i16;
let _c = -150 + _b - _b;
let _d = -150i16;
```
当然可以加下划线 `-150_i16`

### All the Numeric Types  所有数字类型
### Booleans and Characters 布尔值和字符

`bool`  和 `char`

一个 C 语言字符通常只占用一个字节，而一个孤立的 Rust 字符占用四个字节

这是因为 Rust 字符是 Unicode 字符，而 Unicode 标准定义了超过一百万个可能的值。
文字字符用单引号括起来，它们也可以是非 ASCII 字符
```rust
let e_grave = 'è';
let japanese_character = 'さ';
println!("{} {}", e_grave, japanese_character);
```
请注意，与 C 语言不同，bool 和 char 都不是数字。因此，以下两种说法都是非法的
``` rust
let _a = 'a' + 'b';
let _b = false + true;
```
但是，这两种类型都可以转换为数字
``` rust
print!("{} {} {} {} {}", true as u8, false as u8,
    'A' as u32, 'à' as u32, '€' as u32); // 1 0 65 224 8364
```

但是你不能对数字使用‘as bool’子句

char的情况也差不多，不是每一个数字都对应一个char， 也不能用`as char`. 可以使用`char::from_u32`
但是，对于 0 到 255 之间的每个数字，都有一个 Unicode 字符与之对应，因此允许将任意数量的 u8 类型转换为字符。

``` rust
for n in 32..127 {
	println!("{}: [{}]", n, n as u8 as char);
}
for n in 160..256 {
	println!("{}: [{}]", n, n as u8 as char);
}
```

### The Empty Tuple 空元组
还有另一种原始的、奇怪的类型，它在 Rust 中的名称是“()”，它只是一对括号。这种类型只有一个值，写法和它的类型一样，就是“()”。这种类型在某种程度上对应于 C 语言的“void”类型，或 JavaScript 的“未定义”类型，因为它表示没有类型信息。它被命名为空元组。

``` rust
let a: () = ();
let b = { 12; 87; 283 }; 
let c = { 12; 87; 283; };
let d = {};
let e = if false { };
let f = while false { };
print!("{:?} {} {:?} {:?} {:?} {:?}",
    a, b, c, d, e, f);
// () 283 () () () ()
```
`;` 语句
块的值被定义为其最后一个表达式的值，如果存在这样的表达式的话；

### Array and Vector Types  数组和向量类型

明确数组和向量的类型

```rust
let _array1: [char; 3] = ['x', 'y', 'z'];
let _array2: [f32; 200] = [0f32; 200];
let _vector1: Vec<char> = vec!['x', 'y', 'z'];
let _vector2: Vec<i32> = vec![0; 5000];
```

### Constants 常数

``` rust 
let n = 20;
let _ = [0; n]; // 这个是非法的，

const N: usize = 20;
let _ = [0; N]; // 但是这个常数 就是正确的
```
Rust 常量对应于没有参数的 C 语言宏。与前面的 Rust 代码对应的 C 代码是这样的
``` c
#define N 20
int _[N];
```
Rust 常量可以被认为是在编译时与值相关联的名称，而不是与对象相关联的名称。编译器会在程序中使用常量名称的每个地方替换这个值

### Discovering the Type of an Expression (发现表达式的类型)

想知道一个表达式是什么类型，我们可以这样,  定义一个空元组
``` rust 
let _: () = 4u32 / 3u32;
```
这个程序的编译产生了错误不匹配的类型，错误信息的细节解释了预期的`()`, found `u32`。从这个解释中，我们得知我们的表达是 u32 类型的
有时，错误信息比较模糊：
``` rust
let _: () = 4 / 3;
```

对于这个程序，explanation is expected `()`, found integer，`integer`词整数并不表示具体类型；它表示一个仍然不受约束的类型，编译器已经确定它是一个整数类型，但还不是几个现有的整数类型中的哪一个。


