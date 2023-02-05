在本章中，挑战是编写 cat 程序的克隆，之所以这样命名是因为它可以将多个文件连接成一个文件。也就是说，给定文件 a、b 和 c，您可以执行 cat a b c > all 以流式传输这三个文件中的所有行，并将它们重定向到一个名为 all 的文件中。该程序将接受一个选项，在每一行前加上行号。
在本章中，您将学习：
* 如何将代码组织到一个库和一个二进制包中
* 如何使用测试优先开发
* 公有和私有变量和函数的区别
* 如何测试文件是否存在
* 如何为不存在的文件创建随机字符串
* 如何读取常规文件或 STDIN（发音为 standard in）
* 如何使用eprintln！打印到 STDERR 和format!格式化一个字符串
* 如何编写在 STDIN 上提供输入的测试
* 如何以及为什么创建结构
* 如何定义互斥参数
* 如何使用迭代器的枚举方法
* 有关如何以及为何使用 Box 的更多信息

# cat 如何工作

我将从展示 cat 的工作原理开始，以便您了解挑战的预期内容。BSD 版的 cat 没有响应 --help，所以我必须使用 man cat 来阅读手册页。对于这样一个有限的程序，它有惊人数量的选项：

-b 从1开始对非空输出行进行编号.
-n 从1开始对输出行进行编号.

GNU 版本确实响应 --help:

-b, --number-nonblank    number nonempty output lines, overrides -n
-n, --number             number all output lines

对于挑战程序，我将只执行选项 -b|--number-nonblank 和 -n|--number。我还将展示如何在给定文件名参数“-”时读取常规文件和 STDIN。我已经将四个用于测试的文件放03_catr/tests/inputs 目录中：

1. empty.txt：一个空文件
2. fox.txt：单行文本
3. spiders.txt：三行文字
4. the-bustle.txt：艾米丽狄金森的一首可爱的诗，有九行，包括一个空白

空文件很常见，即使没有用。
我包含它是为了确保我的程序可以优雅地处理意外输入。
也就是说，我希望我的程序至少不会倒下。
以下命令不产生任何输出，所以我希望我的程序做同样的事情：

-n|--number 和 -b|--number-nonblank 标志都会对行进行编号，行号在六个字符宽的字段中右对齐，后跟一个制表符，然后是文本行。为了区分制表符，我可以使用-t选项来显示非打印字符，使制表符显示为^I。在以下命令中，我使用 Unix 管道 |将第一个命令的 STDOUT 连接到第二个命令中的 STDIN：
```sh
cat -n tests/inputs/fox.txt | cat -t
1^IThe quick brown fox jumps over the lazy dog.
```

spiders.txt 文件有三行文本，应使用 -b 选项编号：


当处理任何不存在或无法打开的文件时，cat 将向 STDERR 打印一条消息并移动到下一个文件。在以下示例中，我将 blargh 用作不存在的文件。我使用 touch 命令创建文件 cant-touch-this 并使用 chmod 命令设置文件权限使其不可读。当您编写 ls 的克隆时，您将在第 15 章中了解更多关于 000 的含义:

```sh
$ touch cant-touch-this && chmod 000 cant-touch-this
$ cat tests/inputs/fox.txt blargh tests/inputs/spiders.txt cant-touch-this
The quick brown fox jumps over the lazy dog. 
cat: blargh: No such file or directory 
Don't worry, spiders, 
I keep house
casually.
cat: cant-touch-this: Permission denied 
```
“最后，对所有文件运行 cat 并注意它开始为每个文件重新编号行
```sh
$ cat -n empty.txt fox.txt spiders.txt the-bustle.txt
     1	The quick brown fox jumps over the lazy dog.
     1	Don't worry, spiders,
     2	I keep house
     3	casually.
     1	The bustle in a house
     2	The morning after death
     3	Is solemnest of industries
     4	Enacted upon earth,—
     5
     6	The sweeping up the heart,
     7	And putting love away
     8	We shall not want to use again
     9	Until eternity.

```

如果你查看 mk-outs.sh 脚本，你会看到我使用所有这些文件单独和一起执行 cat，作为常规文件并通过 STDIN，不使用标志并使用 -n 和 -b 标志。我将所有输出捕获到 tests/expected 目录中的各种文件，以用于测试。


# 开始测试驱动开发

在第 2 章中，我在本章末尾编写了测试，因为我需要向您展示该语言的一些基础知识。
从本练习开始，我想让您思考 Kent Beck 所著同名书中所描述的测试驱动开发 (TDD)（Addison-Wesley，2002 年）。TDD 提倡在编写代码之前为代码编写测试，如图 3-1 所示。

对于 cat 的 Rust 版本，你编写的挑战程序应该称为 catr（发音为 cat-er）。
我建议您从 cargo new catr 开始一个新的应用程序，然后将我的 03_catr/tests 目录复制到您的源代码树中。除了测试之外不要复制任何东西，因为您将自己编写其余代码。你应该有这样的结构：
```sh
$ tree -L 2 catr/
catr
├── Cargo.toml
├── src
│   └── main.rs
└── tests
    ├── cli.rs
    ├── expected
    └── inputs

4 directories, 3 files
```
我将使用与第 2 章中相同的所有外部箱子加上 rand cate进行测试，因此请将您的 Cargo.toml 更新为：

```
[dependencies]
clap = "2.33"

[dev-dependencies]
assert_cmd = "1"
predicates = "1"
rand = "0.8”
```

现在运行 cargo test 下载crates，编译你的程序，然后运行测试。所有测试都应该失败。如果你选择接受，你的任务就是编写一个能够通过这些测试的程序。

## Creating a Library Crate 

第 2 章中的程序非常短，很容易放入 src/main.rs 中。您在职业生涯中编写的典型程序可能会更长。从这个程序开始，我将把我的代码分成 src/lib.rs 中的库和 src/main.rs 中调用库函数的二进制文件。我相信随着时间的推移，这个组织可以更容易地测试和发展应用程序。
首先，我将把所有重要的部分从 src/main.rs 移到 src/lib.rs 中一个名为 run 的函数中。
此函数将返回一种 Result 以指示成功或失败。这类似于第 2 章中的 TestResult 类型别名。
TestResult 总是返回 Ok 变体中的单元类型 ()，但 MyResult 可以返回一个 Ok，它包含我可以使用泛型 T 表示的任何类型：

``` rust
// src/lib.rs
type MyResult<T> = Result<T, Box<Error>>;

pub fn run() -> MyResult<()> {
    println!("Hello world");
    Ok(())
}
```

```rs
// src/main.rs

fn main() {
    if let Err(e) = catr::run() {
        eprintln!("{}", e);
        std::process::exit(1)
    }
}
```

## 定义参数

接下来，我将添加程序的参数，我想引入一个名为 Config 的结构来表示程序的参数。
结构是一种数据结构，您可以在其中定义它将包含的元素的名称和类型。它类似于其他语言中的类定义。在这种情况下，我想要一个结构来描述程序将需要的值，例如输入文件名列表和用于对输出行进行编号的标志。

``` rust
#[derive(Debug)]
pub struct Config {
    files: Vec<String>,
    number_lines: bool,
    number_nonblank_lines: bool,
}

```

要使用结构，我可以创建一个具有特定值的实例。在下面的 get_args 函数草图中，您可以看它通过使用用户的运行时值创建一个新的 Config 来完成。添加 use clap::{App, Arg} 和这个函数到你的 src/lib.rs。尝试自己完成功能，从第 2 章中窃取你能窃取的内容：

这意味着运行函数需要更新以接受配置参数。现在，打印它：

```rust
pub fn run(config: Config) -> MyResult<()> {
    dbg!(config);
    Ok(())
}
```

Use the dbg! (debug) macro to print the configuration.
更新你的 src/main.rs 如下

```rust
fn main() {
    if let Err(e) = catr::get_args().and_then(catr::run) {
        eprintln!("{}", e);
        std::process::exit(1)
    }
}
```

调用 catr::get_args 函数，然后使用 Result::and_then 将 Ok(config) 传递给 catr::run

看看你是否可以让你的程序打印这样的用法：

```sh
$ cargo run --quiet -- --help
catr 0.1.0
Ken Youens-Clark <kyclark@gmail.com>
Rust cat

USAGE:
    catr [FLAGS] <FILE>...

FLAGS:
    -h, --help               Prints help information
    -n, --number             Number lines
    -b, --number-nonblank    Number non-blank lines
    -V, --version            Prints version information

ARGS:
    <FILE>...    Input file(s) [default: -]
```

虽然 BSD 版本允许同时使用 -n 和 -b，但挑战程序应该认为它们是相互排斥的，并且在一起使用时会产生错误：



有了这么多代码，你应该能够在执行 cargo test 时至少通过几个测试。将会有大量输出显示所有失败的测试输出，但不要绝望。您很快就会看到一个完全通过的测试套件。


## 处理文件

现在您已经验证了所有参数，您已准备好处理文件并创建正确的输出。首先修改 src/lib.rs 中的运行函数来打印每个文件名：
```rust
pub fn run(config: Config) -> MyResult<()> {
    for filename in config.files {
        println!("filename: {}", filename)
    }
    Ok(())
}
```

用一些输入文件运行程序。在以下示例中，bash shell 会将文件 glob *.txt 扩展为所有以扩展名 .txt 结尾的文件名：

```sh
$ cargo run -- tests/inputs/*.txt 
tests/inputs/empty.txt
tests/inputs/fox.txt
tests/inputs/spiders.txt
tests/inputs/the-bustle.txt
```

## 打开文件或 STDIN

下一步是尝试打开每个文件名。当文件名为“-”时，我应该打开STDIN；否则，我将尝试打开给定的文件名并处理错误。对于以下代码，您需要将导入扩展为以下内容：

```rust
use std::error::Error;
use std::fs::File;
use std::io::{self, BufRead, BufReader};
```

这一步有点棘手，所以我想提供一个开放的功能供你使用。在下面的代码中，我使用了 match 关键字，它类似于 C 中的 switch 语句。具体来说，我正在匹配文件名是否等于“-”或其他东西，这是使用通配符 _ 指定的：

```rust
fn open(filename: &str) -> MyResult<Box<dyn BufRead>> {
    println!("open {}", filename);
    match filename {
        "-" => Ok(Box::new(BufReader::new(io::stdin()))),
        _ => Ok(Box::new(BufReader::new(File::open(filename)?))),
    }
}
```

该函数将接受文件名并将返回一个错误或一个实现 BufRead 特征的boxed值。

如果 File::open 成功，结果将是一个文件句柄，这是一种读取文件内容的机制。文件句柄和 std::io::stdin 都实现了 BufRead 特性，这意味着值将例如响应 BufRead::lines 函数以生成文本行。请注意，BufRead::lines 将删除任何行结尾，例如 Windows 上的 \r\n 和 Unix 上的 \n。

你再次看到我正在使用一个 Box 创建一个指向堆分配内存的指针来保存文件句柄。
您可能想知道这是否完全必要。我可以尝试不使用 Box 来编写函数：

如果我尝试编译这段代码，我会收到以下错误：

正如编译器所说，BufRead 中没有针对 Sized 特性的实现。如果一个变量没有固定的、已知的大小，那么 Rust 不能将它存储在堆栈上。解决方案是通过将返回值放入一个 Box 中来在堆上分配内存，这是一个已知大小的指针。

前面的开放功能真的很密集。如果您认为这有点复杂，我将不胜感激；但是，它基本上可以处理您将遇到的任何错误。为了证明这一点，将您的run更改为以下内容：

尝试使用以下命令运行您的程序：
1. 一个有效的输入文件
2. 一个不存在的文件
3. 一个不可读的文件

对于最后一个选项，您可以创建一个无法读取的文件，如下所示：

```sh
touch cant-touch-this && chmod 000 cant-touch-this
```

运行你的程序并验证你的代码优雅地打印错误输入文件的错误消息并继续处理有效的文件：

有了这个添加，你应该能够通过`cargo test skips_bad_file`。现在您可以打开并读取有效的输入文件，我希望您自己完成该程序。你能弄清楚如何逐行读取打开的文件吗？从只有一行的 tests/inputs/fox.txt 开始。您应该能够看到以下输出：

```rust
pub fn run(config: Config) -> MyResult<()> {
    // dbg!(config);
    for filename in config.files {
        println!("filename: {}", filename);
        match open(&filename) {
            Err(err) => eprintln!("faild filename {} : {}", filename, err),
            Ok(mut file) => {
                let mut fileStr = String::new();
                file.read_to_string(&mut fileStr)?;
                println!("open {}, {}", filename, fileStr);
            }
        }
    }
    Ok(())
}

```

```sh
$ cargo run -- tests/inputs/fox.txt
The quick brown fox jumps over the lazy dog.
```

确认您可以默认读取 STDIN。在下面的命令中，我将使用 |将第一个命令的 STDOUT 通过管道传输到第二个命令的 STDIN

```sh
$ cat tests/inputs/fox.txt | cargo run
The quick brown fox jumps over the lazy dog.
```

提供文件名“-”时，输出应该相同。在以下命令中，我将使用 bash 重定向运算符 < 从给定的文件名中获取输入并将其提供给 STDIN：

```sh
$ cargo run -- - < tests/inputs/fox.txt
The quick brown fox jumps over the lazy dog.
```

接下来，尝试一个多行的输入文件，并尝试为 -n 的行编号：

```sh
$ cargo run -- -n tests/inputs/spiders.txt
     1	Don't worry, spiders,
     2	I keep house
     3	casually.
```

然后尝试跳过 -b 编号中的空白行：

```sh
$ cargo run -- -b tests/inputs/the-bustle.txt
     1	The bustle in a house
	 2	The morning after death
     3	Is solemnest of industries
     4	Enacted upon earth,—

     5	The sweeping up the heart,
     6	And putting love away
     7	We shall not want to use again
     8	Until eternity.
```

经常运行`cargo test`以查看哪些测试失败了。tests/cli.rs 中的测试与第 2 章类似，但我添加了更多的组织。例如，我在整个板条箱中使用的那个模块的顶部定义了几个常量 &str 值。我使用 ALL_CAPS 名称的通用约定来强调它们在整个板条箱中的作用域或可见的事实：


为了测试程序在给定一个不存在的文件时会死掉，我使用 rand crate 生成一个不存在的随机文件名。对于下面的函数，我将使用 rand::{distributions::Alphanumeric, Rng} 来导入我在这个函数中需要的 crate 的各个部分：


该函数将返回一个字符串，它是一个动态生成的字符串，与我一直使用的 str 结构密切相关。

如果您还没有完全理解前面的代码，请不要担心。我只是展示这个，以便您了解它是如何在 skips_bad_file 测试中使用的：


1. 生成一个不存在的文件的名称。
2. 在 Windows 或 Unix 平台上，预期的错误消息应包括文件名和字符串“os error 2”。
3. 使用错误文件运行程序并验证 STDERR 是否与预期模式匹配。
4. 该命令应该会成功，因为坏文件应该只会生成警告，而不会终止进程。

我创建了一个运行辅助函数来运行带有输入参数的程序，并验证输出是否与 mk-outs.sh 生成的文件中的文本相匹配：


# 解决方案

我希望你发现这是一个有趣且具有挑战性的程序。一步一个脚印地处理复杂的程序很重要。我将向您展示我是如何以这种方式构建我的程序的。

## 读取文件中的行

##  打印行号

接下来，我想为 -n|--number 选项添加行号打印。C 程序员可能熟悉的一种解决方案是这样的：

回想一下，默认情况下，Rust 中的所有变量都是不可变的，因此有必要将 mut 添加到 line_num 中，因为我打算更改它。+= 运算符是一个复合赋值，它将右边的值 1 添加到 line_num 以递增 it1。同样值得注意的是格式语法 {:>6}，它将字段的宽度指示为六个字符，文本右对齐。（您可以使用 < 表示左对齐，使用 ^ 表示居中文本。）这种语法类似于 C、Perl 和 Python 的字符串格式化中的 printf。

如果我运行这个版本的程序，它看起来很不错

虽然这足够有效，但我想指出一个使用 Iterator::enumerate 的更惯用的解决方案。此方法将返回一个元组，其中包含可迭代对象中每个元素的索引位置和值，这是可以产生值直到耗尽的东西：

这将创建相同的输出，但现在代码避免使用可变值。我可以执行 cargo test fox 来运行所有以 fox 开头的测试，我发现三分之二的测试通过了。该程序在 -b 标志上失败，所以接下来我需要处理仅为非空行打印行号。请注意，在此版本中，我还将删除 line_result 并隐藏 line 变量：

# 更进一步

你现在有了一个工作计划，但你不必就此打住。如果您准备迎接额外的挑战，请尝试实现 BSD 和 GNU 版本手册页中显示的其他选项。对于每个选项，使用 cat 创建预期的输出文件，然后展开测试以检查您的程序是否创建了相同的输出。我还建议您查看 bat，它是 cat（“带翅膀”）的另一个 Rust 克隆，以获得更完整的实现。

cat -n 的行数输出类似于 nl，一个“行号过滤器”程序。cat 也有点类似于一次向您显示一页或一整屏文本的程序，所谓的寻呼机喜欢 more 和 less。（more 会向您显示底部带有“More”的文本页面，让您知道您可以继续。显然有人决定变得聪明并命名他们的克隆，但它做同样的事情。）考虑实施这两个项目。阅读手册页，创建测试输出，并复制该项目的想法来编写和测试您的版本。

#  总结

你在本章中取得了很大进步，创建了一个复杂得多的程序。想想你学到了什么：

* 您将代码分成库 (src/lib.rs) 和二进制 (src/main.rs) 包，这样可以更轻松地组织和封装想法。
* 你创建了你的第一个结构，它有点像其他语言中的类声明。这个结构允许你创建一个名为 Config 的复杂数据结构来描述你程序的输入
* 默认情况下，所有值和函数都是不可变的和私有的。您学会了使用 mut 使值可变，并使用 pub 使值或函数公开。
* 你使用了测试优先的方法，在这种方法中，所有的测试甚至在程序编写之前就已经存在。当程序通过所有测试时，您可以确信您的程序符合测试中编码的所有规范。
* 您了解了如何使用 rand crate 为不存在的文件生成随机字符串。
* 你想出了如何从 STDIN 或常规文件中读取文本行。
* 你使用了 eprintln！打印到 STDERR 和格式的宏！动态生成新字符串。
* 你使用了一个 for 循环来访问可迭代对象中的每个元素。
* 你发现 Iterator::enumerate 方法将索引和元素作为元组返回，这对于给文本行编号很有用。
* 您学会了使用指向文件句柄的 Box 来读取 STDIN 或常规文件