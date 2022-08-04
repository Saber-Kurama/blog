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

运行
``` sh
$ cargo run
```
安静 运行
``` sh
$ cargo run --quiet
```
查看 `cargo` 的所有命令
```sh
$ cargo 
```
## Writing and Running Integration Tests 编写和运行集成测试

> 不仅仅是测试的行为，设计测试的行为是已知的最好的错误预防措施之一。创建有用的测试必须进行的思考可以在编码之前发现并消除错误——事实上，测试设计思维可以在软件创建的每个阶段发现和消除错误，从概念到规范，再到设计、编码，其余的。

我希望向您展示的很大一部分是测试代码的价值。虽然这个程序很简单，但仍有一些事情需要验证。我使用了几大类测试，我可以将它们描述为由内而外，我为程序内部的函数编写测试，以及由外而内，我编写的测试可以像用户一样运行我的程序。第一种测试通常称为单元测试，因为函数是编程的基本单元。我将在第 2 章中向您展示如何编写这些内容。对于这个程序，我想从后一种测试开始，这种测试通常被称为集成测试，因为它检查程序是否作为一个整体工作.

为集成测试代码创建一个测试目录是很常见的。这可以使您的源代码井井有条，并且还可以使编译器在您不进行测试时轻松忽略此代码

``` sh
$ mkdir tests
```

``` sh
“$ tree -L 2
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
├── target
│   ├── CACHEDIR.TAG
│   └── debug
└── tests
    └── cli.rs
```

首先添加这个显示 Rust 测试基本结构的函数：

``` rust
#[test]
fn runs() {
  let mut cmd = Command::new("hello");
  let res = cmd.output();
  assert!(res.is_ok())
}

```
查看 `PATH`

```shell
echo $PATH  | tr : '\n'
```

``` shell
/opt/homebrew/bin
/Users/kyclark/.cargo/bin
/Users/kyclark/.local/bin
/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin
```
即使到那个`target/debug`目录下依然找不到 `hello`

依然使用相对路径
```
$ ./hello
Hello, world!
```

### Adding a Project Dependency  添加项目依赖

目前，`hello`程序只存在于`target/debug`目录下。如果我将它复制到我的 PATH 中的任何目录（注意我包含私有程序的 `$HOME/.local/bin `目录），我可以执行它并成功运行测试。我不想复制我的程序来测试它； 相反，我想测试当前 crate 中的程序。我可以使用 crate `assert_cmd` 在我的 crate 目录中找到该程序。我首先需要将此作为`开发依赖项`添加到 `Cargo.toml`。这告诉 Cargo 我只需要这个 crate 来进行测试和基准测试：

``` toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"


[dependencies]

[dev-dependencies]
assert_cmd = "1"
```
然后我可以使用这个 crate 创建一个查看 Cargo 二进制目录的命令。下面的测试并不能验证程序是否产生了正确的输出，只是它看起来成功了。用这个定义更新你的运行函数：
``` rust
use assert_cmd::Command; 

#[test]
fn runs() {
  let mut cmd = Command::cargo_bin("hello").unwrap();
  cmd.assert().success();
}
```

Import assert_cmd::Command.
创建一个命令以在当前 crate 中运行 hello。这将返回一个 Result，我认为它对 Result::unwrap 是安全的，因为找到了二进制文件。
执行程序并使用 Assert::success 来“确保命令成功

### Understanding Program Exit Values 了解程序退出值

程序成功退出意味着什么？系统程序应向操作系统报告最终退出状态。可移植操作系统接口 (POSIX) 标准规定标准退出代码为 0 表示成功（认为零错误），否则为 1 到 255 之间的任何数字。我可以使用 bash shell 和 true 命令向您展示这一点。
这是 macOS 上存在的版本的 man true 手册页

正如文件显示的那样，这个程序没有成功。如果我运行 true，我什么也看不到，但我可以检查 bash 变量 $? 查看最新程序的退出状态：

```
$ true
$ echo $?
0
```

错误命令是一个必然结果，因为它“总是以非零退出代码退出”：

```
$ false
$ echo $?
1
```
您将在本书中编写的所有程序都将在正常终止时返回值 0，而在出现错误时返回非零值。我可以编写我自己的 true 和 false 版本来向您展示如何做到这一点。首先创建一个 src/bin 目录，然后使用以下内容创建 src/bin/true.rs

```
$ cat src/bin/true.rs
fn main() {
    std::process::exit(0); 
}
```

Use the std::process::exit function to exit the program with the value 0.

You can run the program and manually check the exit value:

```
$ cargo run --quiet --bin true 
$ echo $?
0
```
--bin 选项是“要运行的 bin 目标的名称"

默认情况下，Rust 程序将以代码 0 退出。回想一下 src/main.rs 没有显式调用 std::process::exit。这意味着真正的程序什么都不做

想确定吗？将 src/bin/true.rs 更改为以下内容：

```
$ cat src/bin/false.rs
fn main() {
    std::process::exit(1); 
}
```

Another way to write this program uses `std::process::abort`:
```
$ cat src/bin/false.rs
fn main() {
    std::process::abort();
}
```

再次，运行测试套件以确保程序仍按预期运行。

###  Testing the Program Output 测试程序输出

虽然很高兴知道我的 hello 程序正确退出，但我想确保它实际上将正确的输出打印到 STDOUT（发音为标准输出），这是输出出现的标准位置，通常是控制台。
将 tests/cli.rs 中的运行函数更新为以下内容：

``` rust
#[test]
fn runs() {
  let mut cmd = Command::cargo_bin("hello").unwrap();
  cmd.assert().success().stdout("Hello,world!\n");
}
```

``` rust
fn main() {
    println!("Hello, world!");
}
```

### Exit Values Make Programs Composable 退出值使程序可组合
