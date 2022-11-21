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

编译器建议我应该使用 {:?} 而不是 {} 作为占位符。这是打印结构的调试版本的指令，它将“在面向程序员的调试上下文中格式化输出。”再次参考查看 Args 文档，可以看到 Debug 列在“Trait Implementations”下，因此有效：

``` rust
fn main() {
    println!("{:?}", std::env::args());
}

```

```
❯ cargo run
Args { inner: ["target/debug/echor"] }
```

如果您不熟悉命令行参数，第一个值通常是程序本身的路径。它本身不是一个参数，但它是有用的信息。需要注意的是，这个程序被编译成path target/debug/echor。除非你另有说明，否则 Cargo 将放置可执行文件（也称为二进制文件）的位置。接下来，我将传递一些参数：

```
> cargo run hello world
Args { inner: ["target/debug/echor", "hello", "world"] }
```

万岁！看来我能够为我的程序获取参数。我传递了两个参数，Hello 和 world，它们在二进制名称之后显示为附加值。我知道我需要传递 -n 标志，所以让我试试看：

``` shell
❯ cargo run hello world -n
Args { inner: ["target/debug/echor", "hello", "world", "-n"] }
```

将标志放在值之前也很常见，所以让我试试看：

```
❯ cargo run -n hello world
error: Found argument '-n' which wasn't expected, or isn't valid in this context

        If you tried to supply `-n` as a value rather than a flag, use `-- -n`

USAGE:
    cargo run [OPTIONS] [--] [args]...

For more information try --help
```

这不起作用，因为 Cargo 认为 -n 参数是针对它自己的，而不是我正在运行的程序。要解决这个问题，我需要使用两个破折号分隔 Cargo 的选项：

```
❯ cargo run -- -n hello world

Args { inner: ["target/debug/echor", "-n", "hello", "world"] }
```

在程序参数的说法中，-n 是一个可选参数，因为您可以将其省略。通常程序选项以一个或两个短划线开头。短名称通常带有一个短划线和一个“字符，例如 -h 表示帮助标志和带有两个破折号和一个词的长名称 --help。具体来说，-n 和 -h 是标志，存在时具有一种含义，不存在时具有相反的含义。在这种情况下，-n 表示省略尾随换行符；否则，正常打印

echo 的所有其他参数都是位置参数，因为它们相对于程序名称（参数中的第一个元素）的位置决定了它们的含义。考虑带有两个位置参数的命令 chmod，一个模式如 755 首先是一个文件或目录name second.在 echo 的情况下，所有位置参数都被解释为要打印的文本，并且它们应该按照给定的相同顺序打印。这不是一个糟糕的开始，而是本书中程序的参数将变得更加复杂。
我需要找到一种更好的方法来解析程序的参数。