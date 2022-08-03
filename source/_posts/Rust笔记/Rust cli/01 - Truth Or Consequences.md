## Chapter 1. Truth Or Consequences
在这一章中，我将向你展示如何组织、运行和测试一个Rust程序。
我将使用Unix平台(macOS)来解释关于系统程序的一些基本思想。
这些想法中只有一部分适用于Windows操作系统，但不管你使用哪个平台，Rust程序本身都能工作。
你将学习:
* 如何将Rust代码编译成可执行文件
* 如何使用Cargo启动新项目
* 关于$PATH环境变量
* 如何使用crates.io外部Rust板条箱
* 关于程序的退出状态
* 如何使用常见的系统命令和选项
* 如何编写Rust版本的true 和 false程序
* 如何组织、编写和运行测试

## Getting Started with “Hello, world!”
```rs
fn main () {
  println!("hello world");
}

```

 println! 是一个宏
执行代码之前 需要先编译

```
$ rustc hello.rs
```
用 `file` 命令查看编译后的文件类型

```
❯ file hello
hello: Mach-O 64-bit executable x86_64
```

mac 上直接执行

```
$ ./hello 
Hello, world!
```
在windows上
```
> .\hello.exe
Hello, world!
```

## Organizing a Rust Project Directory 组织一个rust项目目录
在你的 Rust 项目中，你可能会编写许多源代码文件，并且还会使用来自 crates.io 等来源的代码。最好创建一个目录来包含所有这些。我将创建一个名为 hello 的目录，并在其中为 Rust 源代码文件创建一个 src 目录：

```
$ tree
.
├── hello
└── src
    └── hello.rs”

```

## Creating and Running a Project with Cargo 使用 Cargo 创建和运行项目
创建项目
``` sh
$ cargo new hello
```

目录结构
``` sh
$ cd hello
$ tree
.
├── Cargo.toml
└── src
    └── main.rs”

```