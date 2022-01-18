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
```rust
print!("{} ", std::mem::size_of::<i32>()); // 4
print!("{} ", std::mem::size_of_val(&12)); // 4
```
