在第 1 章中，您编写的程序始终以相同的方式运行。
在本章中，您将学习如何使用用户的参数在运行时更改程序的行为。
您将在本章中编写的挑战程序是 echo 的克隆，它将在命令行上打印其参数，可以选择以换行符终止。

在本章中，您将学习:

* 如何使用clap crate处理命令行参数
* 关于STDOUT和STDERR
* 关于 Rust 类型，如字符串、向量、切片和单元类型
* 如何使用匹配、if 和返回等表达式
* 如何使用 Option 表示可选值
* 如何使用 Result 的变体Ok 和 Err处理错误
* 如何退出程序并发出成功或失败的信号
* 堆栈内存和堆内存之间的区别

## 用cargo启动一个新的二进制程序

这个挑战项目将被称为 echo for echo plus r for Rust。
（我不能决定我是把这个念成 eh-core 还是 eh-koh-ar。）
我希望你已经创建了一个目录来保存你将在本书中编写的所有项目。
切换到该目录并使用 Cargo 启动你的 crate，它是一个 Rust 二进制程序或库：

```
$ cargo new echor
     Created binary (application) `echor` package”

```

你应该看到一个熟悉的目录结构

```sh
❯ tree
.
└── echor
    ├── Cargo.toml
    └── src
        └── main.rs

2 directories, 2 files
```

Go into the new echor directory and use Cargo to run the program:

```
❯ cd echor
❯ cargo run
   Compiling echor v0.1.0 (/Users/saber/coding/rust/command-line-rust/02_echor/echor)
    Finished dev [unoptimized + debuginfo] target(s) in 1.98s
     Running `target/debug/echor`
Hello, world!
```

你已经在第 1 章中看到了这个程序，但我想指出关于代码的更多内容:

```
❯ cat src/main.rs
fn main() {
    println!("Hello, world!");
}
```
正如您在第 1 章中看到的，Rust 将通过执行 src/main.rs 中的 main 函数来启动程序。
函数的任何参数都包含在函数名称后的括号中，因此此处的空括号表示该函数不带参数。
所有函数都返回一个值，返回类型可以用箭头和类型表示，例如 -> u32 表示函数返回一个无符号的 32 位整数。
main 没有任何返回类型意味着该函数返回 Rust 所称的单元类型。
请注意，println！宏会自动在输出中附加一个换行符，这是我在用户请求没有终止换行符时需要控制的功能。

## echo如何工作

echo程序非常简单。
我将回顾它的工作原理，以便您可以考虑您的版本需要的所有功能。
首先，echo 会将其参数打印到 STDOUT:
``` shell
❯ echo hello
hello
```
我使用的 bash shell 假定任意数量的空格分隔参数，因此必须用引号括起带有空格的参数。
在以下命令中，我提供了四个词作为一个参数：

``` shell
❯ echo "Rust has assumed control"
Rust has assumed control
```

没有引号，我提供了四个独立的论点。请注意，我在提供参数时使用了不同数量的空格，但 echo 在每个参数之间使用一个空格打印它们：

``` shell
> echo Rust  has assumed   control
Rust has assumed control
```

如果我想保留空格，我必须用引号将它们括起来：

``` shell
> echo "Rust  has assumed   control"
Rust  has assumed   control
```

命令行程序响应标志 -h 或 --help 以打印有关如何使用该程序的消息，即所谓的用法，因为这通常是输出的第一个词，这是非常常见的。如果我尝试用echo这样做，我就得到了标志的文字：

``` shell
❯ echo --help
--help
```

要了解有关该程序的更多信息，请执行 `man echo` 以阅读手册页。您会看到我使用的是 2003 年的 BSD 版本的程序

默认情况下，打印在命令行上的文本以换行符结束。如前面的手册页所示，echo 有一个选项 -n 将省略换行符。根据您的 echo 版本，这可能似乎不会影响输出。例如，我使用的 BSD 版本显示如下：

```
❯ echo -n hello
hello%      
```

无论哪个版本的 echo，我都可以使用 bash 重定向运算符 > 将 STDOUT 发送到文件：

``` shell
$ echo Hello > hello
$ echo -n Hello > hello-n
```

diff 工具将显示两个文件之间的差异。此输出显示第二个文件 (hello-n) 末尾没有换行符：

```shell
> diff hello hello-n
1c1
< Hello
---
> Hello
\ No newline at end of file
```

## Getting Command-Line Arguments 获得命令行参数

首要任务是获取打印的命令行参数。在 Rust 中，您可以为此使用 `std::env::args`。std 告诉我这是在标准库中，并且是普遍有用的 Rust 代码它包含在语言中。env 部分告诉我这是为了与环境交互，这是程序将在其中找到参数的地方。如果您查看该函数的文档，您会看到它返回 **Args** 类型的东西:

``` rust
pub fn args() -> Args
```

如果你点击 [Args 文档](https://doc.rust-lang.org/stable/std/env/struct.Args.html)的链接，你会发现它是一个结构，这是 Rust 中的一种数据结构。如果你沿着页面的左侧看，你会看到诸如特征实现、其他相关结构、函数等内容。我稍后会探讨这些想法，但现在，只需浏览文档并尝试吸收你所看到的。

Edit src/main.rs to print the arguments.I can call the function by using the full path followed by an empty set of parentheses:

``` rust
fn main() {
    println!(std::env::args()); // This will not work
}
```

该编译器消息中有很多信息。首先，关于特性 std::fmt::Display 没有为 Args 实现。Rust 中的特性是一种以抽象方式定义对象行为的方法。如果一个对象实现了 Display 特性，那么它可以被格式化为“面向用户的输出”。再次查看 Args 文档的“特性实现”部分，注意那里确实没有提到 Display。

