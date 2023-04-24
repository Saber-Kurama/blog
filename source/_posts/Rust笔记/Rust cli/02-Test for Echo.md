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

没有引号，我提供了四个独立的论 然后我发名字点。请注意，我在提供参数时使用了不同数量的空格，但 echo 在每个参数之间使用一个空格打印它们：

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

### Adding clap as a Dependency 添加 clap 作为依赖项

尽管有各种方法和 crate 用于解析命令行参数，但我将在本书中专门使用 **clap crate**。开始之前，我需要告诉 Cargo 我想下载这个 crate 并在我的项目中使用它。我可以通过向 Cargo.toml 添加依赖项来执行此操作。编辑您的文件以将 clap 版本 2.33 添加到 [dependencies] 部分：

```
❯ cat Cargo.toml
[package]
name = "echor"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
clap = "2.33"
```


> 
版本“2.33”表示我想使用这个版本。我可以只使用“2”来表示我可以使用“2.x”行中的最新版本。还有很多其他方法可以指示版本，我建议您阅读如何指定依赖项。
> 

下次我尝试构建程序时，Cargo 将下载 clap 源代码（如果需要）及其所有依赖项。例如，我可以运行 cargo build 来构建新的二进制文件而不运行它：

```

```

您可能很好奇这些包去了哪里。Cargo 将下载的源代码放入 $HOME/.cargo，构建工件进入 target/debug/deps 目录。这带来了构建 Rust 项目的一个有趣的部分：每个程序你build 可以使用不同版本的 crates，并且每个程序都构建在一个单独的目录中。如果您曾经像 Perl 和 Python 一样使用共享模块，那么您会很感激您不必担心冲突一个程序需要一些旧的晦涩版本，另一个需要 GitHub 中最新的前沿版本。当然，Python 提供虚拟环境来解决这个问题，其他语言也有类似的解决方案。不过，我发现 Rust 的方法非常令人欣慰。

Rust 将依赖项放入目标目录的一个结果是它现在非常大。
我已经接近 30MB 的项目目录，几乎所有这些都位于目标目录中。
如果您运行 cargo help，您将看到 clean 命令将删除目标目录，代价是将来必须再次重新编译。
如果您暂时不打算处理该项目，则可以回收磁盘空间。

### 使用 clap 解析命令行参数

要学习如何使用 clap 来解析参数，我需要阅读文档。我喜欢使用 [Docs.rs](https://docs.rs/about)，“Rust 编程语言包的开源文档主机”。如何创建一个新的 App 结构。
将您的 src/main.rs 更改为以下内容：

``` rust
use clap::App;

fn main() {
    let _matches = App::new("echor")
        .version("0.1.0")
        .author("saber ---- saber")
        .about("Rust echo")
        .get_matches();
}
```

有了这段代码，我可以使用 -h 或 --help 标志运行程序以获取使用文档。请注意，我不必定义此参数，因为 clap 为我做了这个：

```
❯ cargo run -- -h
 
echor 0.1.0
saber ---- saber
Rust echo

USAGE:
    echor

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information
```

除了帮助标志外，我看到 clap 还自动处理标志 -V 和 --version 以打印程序的版本：

```
❯ cargo run -- -V
    Finished dev [unoptimized + debuginfo] target(s) in 0.10s
     Running `target/debug/echor -V`
echor 0.1.0
```

接下来，我需要定义参数，我可以通过将 `Arg` 结构添加到 `App `来完成这些操作。

``` rust
use clap::{App, Arg};

fn main() {
    let _matches = App::new("echor")
        .version("0.1.0")
        .author("saber ---- saber")
        .about("Rust echo")
        .arg(
            Arg::with_name("text")
                .value_name("TEXT")
                .help("Input text")
                .required(true)
                .min_values(1),
        )
        .arg(
            Arg::with_name("omit_newline")
                .help("Do not print newline")
                .takes_value(false)
                .short("n"),
        )
        .get_matches();

    println!("{:#?}", matches);
}

```


创建一个名称text的 Arg。这是必需的位置参数，必须至少出现一次并且可以重复。

创建一个名为 omit_newline 的新 Arg。这是一个只有短名称 -n 且没有任何值的标志

>   
之前我使用 {:?} 来格式化参数的调试视图。在这里，我使用 {:#?} 来包含换行符和缩进，以帮助我阅读输出。这被称为漂亮打印，因为它更漂亮

```
❯ cargo run -- -h

echor 0.1.0
saber ---- saber
Rust echo

USAGE:
    echor [FLAGS] <TEXT>...

FLAGS:
    -h, --help       Prints help information
    -n               Do not print newline
    -V, --version    Prints version information

ARGS:
    <TEXT>...    Input text
```

-n 标志省略换行 是可选的
所需的输入文本是一个或多个位置参数

使用一些参数运行程序并检查参数的结构：

```
❯ cargo run -- -n Hello world

ArgMatches {
    args: {
        "text": MatchedArg {
            occurs: 2,
            indices: [
                2,
                3,
            ],
            vals: [
                "Hello",
                "world",
            ],
        },
        "omit_newline": MatchedArg {
            occurs: 1,
            indices: [
                1,
            ],
            vals: [],
        },
    },
    subcommand: None,
    usage: Some(
        "USAGE:\n    echor [FLAGS] <TEXT>...",
    
```

如果不带参数运行该程序，您将收到一条错误消息，指出您未能提供所需的参数：

```
❯ cargo run

error: The following required arguments were not provided:
    <TEXT>...

USAGE:
    echor [FLAGS] <TEXT>...

For more information try --help
```

这是一个错误，因此您可以检查退出值以验证它不是 0：

```shell
❯ echo $?
1
```

如果您尝试提供任何未定义的参数，它将触发错误和非零退出值：

> 您可能想知道这种神奇的事情是如何发生的。为什么程序停止并报告这些错误？如果你阅读 App::get_matches 的文档，你会看到“在解析失败时，将向用户显示错误，并且进程将以适当的错误代码退出。

输出中还发生了另一件根本不明显的微妙事情。用法和错误消息都出现在 STDERR（发音为标准错误）上，这是 Unix 输出的另一个通道。要在 bash shell 中看到这一点，我可以将通道 1 (STDOUT) 重定向到名为 out 的文件，将通道 2 (STDERR) 重定向到名为 err 的文件：

```
 cargo run 1>out 2>err
```

你应该看不到该命令的结果，因为所有输出都被重定向到文件。out 文件应该是空的，因为没有任何内容打印到 STDOUT，但 err 文件应该包含 Cargo 的输出和程序的错误消息：

所以你会看到，行为良好的系统程序的另一个标志是将常规输出打印到 STDOUT 并将错误消息打印到 STDERR。有时错误严重到你应该停止程序，但有时它们应该在运行过程中被注意到.例如，在第 3 章中，您将编写一个处理输入文件的程序，其中一些文件将有意不存在或不可读。
我将向您展示如何向 STDERR 打印关于这些文件的警告并跳到下一个参数。

## Creating the Program Output 创建程序输出

我的下一步是使用用户提供的值来创建程序的输出。将匹配项中的值复制到变量中是很常见的。
首先，我想提取文本参数。因为这个 Arg 被定义为接受一个或多个值，所以我可以使用这些返回多个值的函数之一：

* ArgMatches::values_of: returns `Option<Values>`
* ArgMatches::values_of_lossy: returns `Option<Vec<String>>`



要决定使用哪个，我必须跑几个例子来理解以下概念：

* Option：一个值为 None 或 Some(T) 的值，其中 T 是任何类型，如字符串或整数。在 ArgMatches::values_of_lossy 的情况下，类型 T 将是一个字符串向量。
* Values：用于从参数中获取多个值的迭代器。
* Vec： 一个向量，它是一个连续的可增长数组类型。
* String： 一串字符

这两个函数 ArgMatches::values_of 和 A​​rgMatches::values_of_lossy 都会返回一个Option。由于我最终想要打印字符串，因此我将使用 ArgMatches::values_of_lossy 函数来获取 `Option<Vec<String>>`，Option::unwrap 函数将从 Some(T) 中取出值以获取重载 T。
因为 text 参数是 clap 所必需的，所以我知道 None 是不可能的；因此，我可以安全地调用 Option::unwrap 来获取` Vec<String>` 值

```rust
let text = matches.values_of_lossy("text").unwrap();
```

omit_newline 参数更容易一些，因为它存在或不存在。这个值的类型是 bool 或 Boolean，要么是 true 要么是 false：

```rust
let omit_newline = matches.is_present("omit_newline");
```

最后，我想打印这些值。因为文本是字符串向量，所以我可以使用 Vec::join 将单个空格上的所有字符串连接成一个新字符串以进行打印。在 echor 程序中，clap 将创建向量。为了演示 Vec::join 是如何工作的，我将向您展示如何使用 vec!宏观：
```rust
let text = vec!["Hello", "world"];
```
Vec::join 将在向量的所有元素之间插入给定的字符串以创建一个新字符串。我可以使用 println！将新字符串打印到 STDOUT，后跟一个换行符：
```rust
println!("{}", text.join(" "));
```
在 Rust 文档中，使用断言来呈现事实是常见的做法`assert!`说某事是真的或` assert_eq！`证明一件事等同于另一件事。
在下面的代码中，我可以断言 text.join(" ") 的结果等于字符串“Hello world”：
```rust
assert_eq!(text.join(" "), "Hello world");
```

当存在 -n 标志时，输出应省略换行符。我会改用`print！`不添加换行符的宏，我将根据omit_newline 的值选择添加换行符或空字符串。

```rust
print!("{}{}", text.join(" "), if omit_newline { "" } else { "\n" });
```

这需要我编写一些测试来使用各种输入运行我的程序，并验证它产生与原始 echo 程序相同的输出。

# 集成和单元测试

在第 1 章中，我展示了如何创建从命令行运行程序的集成测试，就像用户将执行的操作一样，以确保程序正常工作。在本章中，我还将向您展示如何编写用于执行单个功能的单元测试，这些功能可能被视为一个编程单元。
首先，您应该将以下依赖项添加到 Cargo.toml。Note that I’m adding `predicates` to this project:

``` yaml
[dev-dependencies]
assert_cmd = "1"
predicates = "1"
```

经常编写测试以确保我的程序在错误运行时失败。例如，如果没有提供任何参数，该程序应该会失败并打印帮助文档。创建一个*tests*目录，然后使用以下内容创建*tests/cli.rs*：

```rust
#[test]
fn dies_no_args() {
    let mut cmd = Command::cargo_bin("echor").unwrap();
    cmd.assert()
        .failure()
        .stderr(predicate::str::contains("USAGE"));
}
```

Import the predicates crate

在没有参数的情况下运行程序并断言它失败并向 STDERR 打印一个用法语句。

> 我通常用前缀 dies 来命名这些类型的测试，这样我就可以用 cargo test dies 来运行它们，以确保程序在各种条件下失败。

我还可以添加一个测试来确保程序在提供参数时成功退出

```rust
#[test]
fn runs() {
    let mut cmd = Command::cargo_bin("echor").unwrap();
    cmd.arg("hello").assert().success();
}
```

### 创建测试输出文件

我现在可以运行 cargo test 来验证我有一个程序可以运行、验证用户输入并打印使用情况。接下来，我想确保程序创建与 echo 相同的输出。首先，我需要捕获各种输入的原始 echo 的输出，以便我可以将它们与我的程序的输出进行比较。在我的 GitHub 存储库的 02_echor 目录中，您会找到一个名为 mk-outs.sh 的 bash 脚本，我用它为各种参数生成 echo 的输出。你可以看到，即使使用这样一个简单的工具，仍然存在相当多的圈复杂度，它指的是所有参数可以组合的各种方式。我需要使用和不使用换行选项检查一个或多个文本参数:

```sh
#!/usr/bin/env bash

OUTDIR="tests/expected"
[[ ! -d "$OUTDIR" ]] && mkdir -p "$OUTDIR"

echo "Hello there" > $OUTDIR/hello1.txt
echo "Hello"  "there" > $OUTDIR/hello2.txt
echo -n "Hello  there" > $OUTDIR/hello1.n.txt
echo -n "Hello"  "there" > $OUTDIR/hello2.n.txt
```

### 比较程序输出

第一个输出文件是用输入 Hello there 作为单个字符串生成的，输出被捕获到文件 tests/expected/hello1.txt 中。对于我的下一个测试，我将使用此参数运行 echor 并将输出与该文件的内容进行比较。确保将 use std::fs 添加到 tests/cli.rs 中，引入标准文件系统模块，然后将 runs 函数替换为：

```rust
#[test]
fn runs() {
    let outfile = "tests/expected/hello1.txt";
    let expected = fs::read_to_string(outfile).unwrap();
    let mut cmd = Command::cargo_bin("echor").unwrap();
    cmd.arg("Hello there").assert().success().stdout(expected);
}
```

使用给定的参数运行程序并断言它成功完成并且 STDOUT 是预期值

> fs::read_to_string 是一种将文件读入内存的便捷方式，但如果您碰巧读取了超出可用内存的文件，它也是一种使您的程序（甚至可能是您的计算机）崩溃的简单方法。您应该只对小文件使用此功能。正如 Ted Nelson 所说：“关于计算机的好消息是，它们会按照您的吩咐去做。坏消息是他们会按照你的吩咐去做。

### 使用Result类型

我一直在以这样一种方式使用 Result::unwrap 方法，即假设每个容易出错的调用都会成功。例如，在 hello1 函数中，我假设输出文件存在并且可以打开并读入一个字符串。在我有限的测试中，情况可能是这样，但做出这样的假设是危险的。我宁愿更加谨慎，所以我将创建一个名为 TestResult 的类型别名。这将是一个特定类型的 Result，它要么是一个总是包含单元类型的 Ok，要么是某个实现 std::error::Error 特性的值：

```rust
type TestResult = Result<(), Box<dyn std::error::Error>>;
```

在前面的代码中，Box 表示错误将存在于一种指针中，其中内存是在堆上动态分配的而不是栈，dyn "用于突出显示对关联 Trait 上的方法的调用是动态调度的。"这真的是很多信息，如果你的眼睛呆滞，我不怪你。简而言之，我是说 TestResult 的 Ok 部分将永远只包含单元类型，而 Err 部分可以包含任何实现 std::error::Error 特性的东西。

这以一些微妙的方式改变了我的测试代码。所有函数现在都表明它们返回一个 TestResult。以前我使用 Result::unwrap 解包 Ok 值并在出现 Err 时恐慌，导致测试失败。在下面的代码中，我将 unwrap 替换为 ?运算符解压缩 Ok 值或将 Err 值传播到返回类型。也就是说，这将导致函数将 Option 的 Err 变体返回给调用者，进而导致测试失败。如果测试函数中的所有代码都成功运行，我将返回一个包含单元类型的 Ok 以指示测试通过。请注意，虽然 Rust 确实有 return 以从函数中提前返回一个值，但习惯用法是省略最后一个表达式中的分号以隐式返回该结果。
将你的 tests/cli.rs 更新为：

```rust
#[test]
fn dies_no_args() -> TestResult {
    let mut cmd = Command::cargo_bin("echor")?;
    cmd.assert()
        .failure()
        .stderr(predicate::str::contains("USAGE"));
    Ok(())
}
```
利用 ？而不是 Result::unwrap 来解包一个 Ok 值或传播一个 Err。

下一个测试传递两个参数，Hello 和 there，并期望程序打印 Hello there。

我总共有四个文件要检查，所以我有必要编写一个辅助函数。我将调用它运行并将参数字符串与预期的输出文件一起传递给它。我不使用向量作为参数，而是使用 std::slice，因为我不需要在定义参数列表后增加它：

```rust
type TestResult = Result<(), Box<dyn std::error::Error>>;

fn run(args: &[&str], expected_file: &str) -> TestResult {
    let expected = fs::read_to_string(expected_file)?;
    let mut cmd = Command::cargo_bin("echor")?;
    cmd.args(args).assert().success().stdout(expected);
    Ok(())
}
```

# 总结

现在你已经在 src/main.rs 中为 echor 程序编写了大约 30 行 Rust 代码，并在 tests/cli.rs 中编写了五个测试来验证你的程序是否满足某种规范（如他们所说的规范）。
想想你取得的成就：
*  行为良好的系统程序应该将基本输出打印到 STDOUT，将错误打印到 STDERR。
* 您已经编写了一个程序，该程序采用选项 -h|--help 来生成帮助，-V|--version 来显示程序的版本，以及 -n 来省略换行符以及一个或多个位置命令行参数。
* 如果程序以错误的参数或 -h|--help 标志运行，它将打印使用文档。
* 该程序将回显所有以空格连接的命令行参数。
* 如果存在 -n 标志，尾随的换行符将被省略
* 你可以运行集成测试来确认你的程序复制了 echo 的输出，至少有四个测试用例覆盖一个或两个输入，无论是否有尾随换行符。
* 你学会了使用几种 Rust 类型，包括单元类型、字符串、向量、切片、选项和结果，以及如何为称为 TestResult 的特定结果类型创建类型别名。
* 你使用了一个 Box 创建了一个指向堆内存的智能指针。这需要深入了解堆栈（变量具有固定的已知大小并按 LIFO 顺序访问）和堆（变量大小可能在程序期间发生变化并通过指针访问）之间的差异。
* 您学习了如何将文件的全部内容读入字符串。
* 您学习了如何从 Rust 程序中执行外部命令、检查退出状态以及验证 STDOUT 和 STDERR 的内容。

所有这一切，而且你是在用一种语言编写的时候完成的，这种语言根本不允许你犯导致错误程序或安全漏洞的常见错误。
当您考虑 Rust 将如何帮助您征服世界时，请随意给自己高五分或享受稍微邪恶的 MWUHAHA 笑声。
现在我已经展示了如何组织和编写测试和数据，我将在下一个程序中更早地使用测试，这样我就可以开始使用测试驱动开发，我先编写测试，然后编写代码来满足测试。












