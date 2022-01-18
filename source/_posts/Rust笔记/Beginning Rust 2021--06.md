---
 title: Rust 2021 版入门
 date: 2022-01-18
 tag: rust
---

# RUST

## Data Implementation 数据实现
*  How to know how many bytes of stack are taken by objects of various types （如何知道各种类型的对象占用了多少字节的堆栈）
* How to shorten the path to access functions declared in external modules （如何缩短访问外部模块中声明的函数的路径）
* How bits are stored in primitive type objects （bits 如何存储在原始类型对象中）
* How to know where an object is stored in memory （如何知道对象在内存中的存储位置）
* Why padding can increase the size taken by some objects （为什么填充可以增加某些对象的大小）
* How vectors are implemented （向量是如何实现的）

### Discovering the Size of Objects （发现Objects的大小）

给定一个源文件，Rust 编译器可以自由生成任何机器代码，只要它以 Rust 语言为该源代码指定的方式运行
因此，例如，给定一个变量，它使用了多少内存位，以及它在内存中的位置都没有定义。编译器甚至可以从内存中删除该变量，因为它从未使用过，或者因为它保存在处理器寄存器中。
然而，看看 Rust 程序使用的数据排列的可能典型实现是有启发性的。
为此，可以使用一些 Rust 功能：
获取字节大小
```rust
print!("{} ", std::mem::size_of::<i32>()); // 4
print!("{} ", std::mem::size_of_val(&12)); // 4
```

### The use Directive 使用指令
如果必须多次指定到达库函数的路径，使用 use 指令将全部或部分路径导入当前范围很方便
前面的例子可以这样改写
``` rust
use std::mem::size_of;
use std::mem::size_of_val;
print!("{} ", size_of::<i32>());
print!("{} ", size_of_val(&12));
```

### The Sizes of the Primitive Types  (原始类型的大小)

``` rust
use std::mem::*;
print!("{} {} {} {} {} {} {} {} {} {} {} {} {} {}",
    size_of::<i8>(),
    size_of::<u8>(),
    size_of::<i16>(),
	size_of::<u16>(),
    size_of::<i32>(),
    size_of::<u32>(),
    size_of::<i64>(),
    size_of::<u64>(),
    size_of::<i128>(),
    size_of::<u128>(),
    size_of::<f32>(),
    size_of::<f64>(),
    size_of::<bool>(),
    size_of::<char>());
// In any computer, this will print: 1 1 2 2 4 4 8 8 16 16 4 8 1 4.
```

### The Representation of Primitive Types (原始类型的表示)

Rust 不鼓励访问对象的内部表示，所以这并不容易；但有一个技巧可以做到这一点：

``` rust
fn as_bytes<T>(o: &T) -> &[u8] {
    unsafe {
        std::slice::from_raw_parts(
            o as *const _ as *const u8,
            std::mem::size_of::<T>())
    }
}
println!("{:?}", as_bytes(&1i8));
println!("{:?}", as_bytes(&2i16));
println!("{:?}", as_bytes(&3i32));
println!("{:?}", as_bytes(&(4i64 + 5 * 256 + 6 * 256 * 256)));
println!("{:?}", as_bytes(&'A'));
println!("{:?}", as_bytes(&true));
println!("{:?}", as_bytes(&&1i8));
```
In an x86_64 system, this could print:
```
[1]
[2, 0]
[3, 0, 0, 0]
[4, 5, 6, 0, 0, 0, 0, 0]
[65, 0, 0, 0]
[1]
[129, 165, 54, 102, 23, 86, 0, 0]
```
通用函数` as_bytes` 使用了一些我们还没有见过的 Rust 结构，这里不会解释，因为理解它的作用不需要他们的知识。它只是简单地引用任何类型的参数，并返回一个对象，该对象表示该对象中包含的字节序列。通过打印这样的对象，您可以将任何对象的表示形式视为它存储在内存中的字节序列
首先，一个值为 1 的 i8 存储在一个字节中。这在任何受支持的硬件架构中都是一样的。
然后，将值为 2 的 i16 存储为一对字节，其中第一个为 2，第二个为 0。这在 32 位和 64 位处理器上都会发生，但仅在所谓的 little -endian 硬件体系结构，即存储多字节数字的硬件体系结构，将最低有效字节放在最低地址。相反，大端硬件架构会打印 [0, 2]。
请注意，char 存储为包含此类字符的 Unicode 值的 32 位数字，bool 存储为单个字节，1 表示真，0 表示假。
最后，最后一条语句打印一个 i8 数字的地址。对于 64 位处理器，这样的地址占用 8 个字节，并且每次运行都不同.

### Location of Bytes in Memory  内存中字节的位置

您还可以发现任何对象的（虚拟）内存位置，即它的地址：

``` rust
let b1 = true;
let b2 = true;
let b3 = false;
print!("{} {} {}",
    &b1 as *const bool as usize,
    &b2 as *const bool as usize,
    &b3 as *const bool as usize);
```

In a 64-bit system, this will print three huge numbers, resembling `140729279188109 ` `140729279188110`  `140729279188111`. 

但是，有一种更简单的方法可以以十六进制表示法打印任何对象的地址:

```rust
let b1 = true;
let b2 = true;
let b3 = false;
print!("{:p} {:p} {:p}", &b1, &b2, &b3);
```
It will print something like:
`0x7ffe16b20c8d` `0x7ffe16b20c8e` `0x7ffe16b20c8f`

### Sizes of Composite Data Types  复合数据类型的大小

当有一系列复合对象时出现填充的效果
``` rust
enum E1 { E1a, E1b }
enum E2 { E2a, E2b(f64) }
use std::mem::*;
print!("{} {} {} {} {} {}",
    size_of_val(&[0i16; 80]), // 80 * 2 = 160
    size_of_val(&(0i16, 0i64)),
    size_of_val(&[(0i16, 0i64); 100]),
    size_of_val(&E1::E1a),
    size_of_val(&E2::E2a),
    size_of_val(&vec![(0i16, 0i64); 100]));
```
This will print: 160 16 1600 1 16 24.


