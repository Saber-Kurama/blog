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