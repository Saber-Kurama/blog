本章的挑战是实现 head 程序，它将打印一个或多个文件的前几行或字节。这是查看常规文本文件内容的好方法，通常是比 cat 更好的选择。当面对某个进程的输出文件之类的目录时，这是快速扫描潜在问题的好方法。
在本练习中，您将学习：
* 如何创建接受值的可选命令行参数
* 如何将字符串解析为数字
* 如何编写和运行单元测试
* 如何使用match臂 护卫
* 如何使用 From、Into 和 as 在类型之间进行转换
* 如何使用 take 在迭代器或文件句柄
* 如何在读取文件句柄时保留行尾
* 如何从文件句柄中读取字节

# head 是如何工作

您应该记住，原始的AT&T Unix操作系统有许多实现，例如BSD（伯克利标准发行版）、SunOS/Solaris、HP-UX和Linux。
这些操作系统中的大多数都有某种版本的头部程序，默认显示一个或多个文件的前十行。
大多数可能会有选项-n来控制显示的行数，-c来显示一些字节数。BSD版本只有这两个选项，我可以通过*main head*看到：

```sh
HEAD(1)                         General Commands Manual                        HEAD(1)

NAME
     head – display first lines of a file

SYNOPSIS
     head [-n count | -c bytes] [file ...]

DESCRIPTION
     This filter displays the first count lines or bytes of each of the specified
     files, or of the standard input if no files are specified.  If count is omitted
     it defaults to 10.

     The following options are available:

     -c bytes, --bytes=bytes
             Print bytes of each of the specified files.

     -n count, --lines=count
```

使用GNU版本，我可以运行`head --help`阅读用法：

请注意，GNU版本能够用负数指定-n和-c，并使用K、M等后缀，我不会实现。在这两个版本中，文件都是可选的位置参数，默认情况下或当文件名为“-”时，将读取STDIN。-N和-b是取整数值的可选参数。

为了演示一些使用head的示例，我将使用04_headr/tests/inputs中找到的文件。给定一个空文件，没有输出，您可以使用头部测试/inputs/empty.txt进行验证。默认情况下，head将打印文件的前10行。如果一个文件少于10行，它将打印所有行。您可以使用tests/inputs/three.txt看到这一点，它有3行：

```shell
❯ head tests/inputs/three.txt
Three
lines,
four words.%  
```

-N选项允许您控制显示的行数。例如，我只能使用以下命令选择2行：

```sh
head -n 2 tests/inputs/three.txt
Three
lines,
```

-C选项仅显示文件中给定的字节数，例如，仅显示前4个字节：

```sh
❯ head -c 4  tests/inputs/three.txt
Thre%       
```
奇怪的是，GNU版本将允许您同时提供-n和-c，并默认显示字节。BSD版本将拒绝这两个参数：

任何不是正整数的-n或-c值都将产生一个错误，该错误将停止程序，该错误将回波非法值：

```sh
❯ head -n 0  tests/inputs/one.txt
head: illegal line count -- 0
❯ head -c foo  tests/inputs/one.txt
head: illegal byte count -- foo
```

当有多个参数时，head会添加一个标题，并在每个文件之间插入一个空行：
```sh
❯ head -n 1  tests/inputs/*.txt
==> tests/inputs/empty.txt <==

==> tests/inputs/one.txt <==
One lines, four words.
==> tests/inputs/three.txt <==
Three

==> tests/inputs/two.txt <==
Two lines,
```

在没有文件参数的情况下，head将从STDIN读取：

```sh
❯ cat tests/inputs/three.txt | head -n 2
Three
lines,
```

与第3章中的`cat`一样，任何不存在或不可读的文件都会被跳过，并打印给STDERR的警告。
在以下命令中，我将使用blargh作为不存在的文件，并将创建一个名为cant-touch-this的不可读文件：

```sh
❯ touch cant-touch-this && chmod 000 cant-touch-this
❯ head blargh cant-touch-this tests/inputs/one.txt
head: blargh: No such file or directory
head: cant-touch-this: Permission denied
==> tests/inputs/one.txt <==
One lines, four words.%        
```

这将与挑战计划预期的重新创建一样多。

## 开始

你可能已经预料到，我想让你写的程序将被称为headr（发音为header）。首先运行`cargo new headr`，并将我的04_headr/tests目录复制到您的项目目录中。将以下依赖项添加到您的Cargo.toml中：

```toml

[dependencies]
clap = "2.33"

[dev-dependencies]
assert_cmd = "1"
predicates = "1"
rand = "0.8"
```

我建议你再次拆分你的源代码，这样src/main.rs看起来像这样：

```rust
fn main() {
    if let Err(e) = headr::get_args().and_then(headr::run) {
        eprint!("{}", e);
        std::process::exit(1);
    }
}
```

通过引入`clip`和`Error`特征并声明MyResult来开始您的src/lib.rs，您可以从第3章的源代码中复制它：

```rust
type MyResult<T> = Result<T, Box<dyn Error>>;
```

该程序将有三个参数，可以用配置结构表示：

原始的usize是“指针大小的无符号整数类型”，其大小从32位操作系统上的4字节到64位操作系统上的8字节不等。Usize的选择有点任意，因为我只想存储某种正整数。我也可以使用u32（无符号32位整数）或u64（无符号64位整数），但我绝对想要一个无符号类型，因为它只表示正整数值。我需要使用像i32或i64这样的有符号整数来表示正数或负数，如果我想像GNU版本那样允许负值，就需要这样做。

行和字节将用于几个函数，其中一个需要usize，另一个需要u64。
这将为稍后讨论如何在类型之间转换提供机会。
您的程序应使用10作为行的默认值，但字节将是一个选项，这是我在第2章中首次介绍的。
这意味着，如果用户提供了有效的值，字节将是`Some<usize>`，如果没有，字节将是None。

您可以使用以下大纲启动get_args函数。您需要添加代码来解析参数并返回Config结构：

``` rust
pub fn get_args() -> MyResult<Config> {
    let matches = App::new("headr")
        .version("0.1.0")
        .author("saber")
        .about("Rust head")
        ...
        .get_matches();
    Ok(Config {})
}
```


此程序的所有命令行参数都是可选的，因为文件将默认为“-”，行将默认为10，并且可以省略字节。第3章中的可选参数是标志，但这里的行和字节需要将Arg::takes_value设置为true。



您可以从打印配置的运行函数开始：

```rust
pub fn run(options: Config) -> MyResult<()> {
    println!("{:#?}", options);
    Ok(())
}

```

### 将字符串解析为数字

Clap返回的所有值都是字符串，但当存在时，您需要将行和字节转换为整数。我将向您展示如何使用str::parse。当提供的值无法解析为数字或包含转换后的数字的Ok时，此函数将返回一个结果，该结果将是一个Err。我将编写一个名为parse_positive_int的函数，该函数试图将字符串值解析为正的usize值。您可以将此添加到您的src/lib.rs中：

```rust
fn parse_positive_int(val: &str) -> MyResult<usize> {
    unimplemented!()
}
```

1. 此函数接受&str，并将返回正usize或错误。
2.  unimplemented!宏“通过惊慌失措地显示未实现的消息来指示未实现的代码。

本着测试驱动开发的精神，我将为此功能添加一个单元测试。我建议在它正在测试的功能之后添加这个：

``` rust
#[test]
fn test_parse_positive_int() {
    // 测试数字字符串3
    let res = parse_positive_int("3");
    assert!(res.is_ok());
    assert_eq!(res.unwrap(), 3);

    // 测试非数字字符串
    let res = parse_positive_int("foo");
    assert!(res.is_err());
    assert_eq!(res.unwrap_err().to_string(), "foo".to_string());

    // ”0“
    let res = parse_positive_int("0");
    assert!(res.is_err());
    assert_eq!(res.unwrap_err().to_string(), "0".to_string());
}
```

运行`cargo test parse_positive_int`，并验证测试是否确实失败。现在停止阅读，写一个通过此测试的函数版本。我会在这里等你完成。

```rust
  match val.parse::<usize>() {
        Ok(v) if v > 0 => Ok(v),
        _ => Err(From::from(val)),
    }
```

怎么样？膨胀，我敢打赌！这是我写的通过上述测试的函数：

1. 尝试解析给定的值。Rust从返回类型中推断出usize类型。
2. 如果解析成功，并且解析值n大于0，则将其作为Ok变体返回。
3. 对于任何其他结果，返回具有给定值的Err。

到目前为止，我已经使用过几次匹配，但这是我第一次展示匹配手臂可以包括后卫，这是模式匹配后的额外检查。我不知道你怎么样，但我觉得这很贴心。

### 将字符串转换为错误

当我无法将给定的字符串值解析为正整数时，我想返回原始字符串，以便将其包含在错误消息中。为了在前面的函数中执行此操作，我使用了冗余的From::from函数将输入&str值转换为错误。考虑一下这个版本，我试图将无法解析的字符串直接放入错误中：

```rust
match val.parse::<usize>() {
        Ok(v) if v > 0 => Ok(v),
        _ => Err(val),
    }
```

如果我尝试编译这个，我会收到以下错误：

```shell
error[E0308]: mismatched types
   --> src/lib.rs:41:18
    |
41  |         _ => Err(val),
    |              --- ^^^ expected struct `Box`, found `&str`
    |              |
    |              arguments to this enum variant are incorrect
    |
    = note: expected struct `Box<dyn std::error::Error>`
            found reference `&str`
note: tuple variant defined here
```

问题是，我应该返回一个MyResult，该结果被定义为任何类型的T类型的`Ok<T>`或实现`Err`特征并存储在`Box`中的东西：

```rust
type MyResult<T> = Result<T, Box<dyn Error>>;
```

在前面的代码中，&str既不实现错误，也不存在于Box中。我可以通过将此更改为`Err（Box::new（val）`来尝试根据建议解决这个问题。
不幸的是，这仍然无法编译，因为我仍然没有满足错误特征：

``` shell
error[E0277]: the trait bound `str: std::error::Error` is not satisfied
  --> src/lib.rs:41:18
   |
41 |         _ => Err(Box::new(val)),
   |                  ^^^^^^^^^^^^^ the trait `std::error::Error` is not implemented for `str`
   |
   = note: required for `&str` to implement `std::error::Error`
   = note: required for the cast from `&str` to the object type `dyn std::error::Error`

```

输入`std::convert::From` trait，这有助于从一种类型转换为另一种类型。
例如，文档展示了如何从str转换为字符串：

```rust
let string = "hello".to_string();
let other_string = String::from("hello");
assert_eq!(string, other_string);
```

就我而言，我可以使用`std::convert::From`和`std::convert::Int`o以多种方式将&str转换为错误。正如文档所述：

在执行错误处理时，From也非常有用。当构造一个能够失败的函数时，返回类型通常以Result<T, E>的形式。From特征通过允许函数返回封装多个错误类型的单个错误类型来简化错误处理。

```rust
 match val.parse::<usize>() {
        Ok(v) if v > 0 => Ok(v),
        // _ => Err(val.into()),
        // _ => Err(Into::into(val)),
        _ => Err(From::from(val)),
    }
```


现在您有办法将字符串转换为数字，请将其集成到您的get_args中。看看你是否可以让你的程序打印以下用法。请注意，我使用GNU版本中的短名称和长名称：

```shell
$ cargo run -- -h
headr 0.1.0
saber saber@qq.com
Rust head

USAGE:
    headr [OPTIONS] <FILES>...

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -c, --bytes <BYTES>    Number of bytes
    -n, --lines <LINES>    Number of lines [default: 10]

ARGS:
    <FILES>...    Input file(s) [default: -]
```

在没有输入的情况下运行程序，并验证默认值是否设置正确：


使用参数运行程序，并确保它们被正确解析：


如果我提供多个位置参数，它们都将进入文件，-c参数将进入字节。在以下命令中，我再次依靠bash shell将文件`glob *.txt`扩展为以.txt结尾的所有文件。PowerShell用户应参考第3章中显示的Get-ChildItem的等效用法：

任何无法解析为正整数的-n或-c值都应导致程序因错误而停止：


程序应该不允许-n和-c一起存在：

解析和验证参数是一个挑战，但我知道你可以做到。当你弄清楚这一点时，请务必参考clap文档。我建议您在您的程序通过`cargo test dies`中包含的所有测试之前不要继续前进：


### 定义参数

以下是我如何定义clap的论点。请注意，行和字节的两个选项将采用值。这与第3章中实现的用作布尔值的标志不同：

```rust
 let matches: clap::ArgMatches = App::new("headr")
        .version("0.1.0")
        .author("saber saber@qq.com")
        .about("Rust head")
        .arg(
            Arg::with_name("lines")
                .short("n")
                .long("lines")
                .value_name("LINES")
                .help("Number of lines")
                .default_value("10"),
        )
        .arg(
            Arg::with_name("bytes")
                .short("c")
                .long("bytes")
                .value_name("BYTES")
                .help("Number of bytes")
                .takes_value(true)
                .conflicts_with("lines"),
        )
        .arg(
            Arg::with_name("files")
                .value_name("FILES")
                .help("Input file(s)")
                .required(true)
                .default_value("-")
                .min_values(1),
        )
        .get_matches();
```


1. 行选项取一个值，默认为“10”。
2.  字节选项接受一个值，它与行参数冲突，因此它们是相互排斥的。
3. 文件参数是位置性的，是必需的，接受一个或多个值，并默认为“-”。  

> Arg::value_name将打印在使用文档中，因此请务必选择一个描述性名称。不要将此与Arg::with_name混淆，Arg::with_name唯一定义了在代码中访问的参数的名称。


以下是我如何在get_args中使用parse_positive_int来验证行和字节。当函数返回Err变体时，我使用?将错误传播到主程序并结束程序；否则，我返回配置：

```rust
let lines = matches
        .value_of("lines")
        .map(parse_positive_int)
        .transpose()
        .map_err(|e| format!("illegal line count -- {}", e))?;
    let bytes = matches
        .value_of("bytes")
        .map(parse_positive_int)
        .transpose()
        .map_err(|e| format!("illegal byte count -- {}", e))?;
    Ok(Config {
        files: matches.values_of_lossy("files").unwrap(),
        lines: lines.unwrap(),
        bytes,
    })
```

``
1. `ArgMatches.value_of`返回一个`Option<&str>`。
2. 使用`Option::ma`p从`Some`解压`&str`并将其发送到`parse_positive_int`。
3. `Option::map`的结果将是`Option<Result>`，`Option::transpose`将其转换为`Result<Option>`。
4. 如果出现错误，请创建一个信息性错误消息。使用？传播错误或解压Ok值。
5. bytes同样处理
6. 文件选项应该至少有一个值，因此应该可以安全地调用Option::unwrap。
7. 这些行具有默认值，并且可以安全地展开。
8. 字节应保留为选项。使用结构字段init速记，因为字段的名称与变量相同。

在前面的代码中，我本可以像这样用每个键/值对编写配置：
Clippy将提出以下建议：

```rust
“$ cargo clippy
warning: redundant field names in struct initialization
  --> src/lib.rs:61:9
   |
61 |         bytes: bytes,
   |         ^^^^^^^^^^^^ help: replace it with: `bytes`
   |
   = note: `#[warn(clippy::redundant_field_names)]` on by default
   = help: for further information visit https://rust-lang.github.io/
     rust-clippy/master/index.html#redundant_field_names”
```

验证所有用户输入需要相当多的工作，但现在我有一些保证，我可以继续处理好的数据。

### 处理输入文件

这个挑战程序应该像第3章一样处理输入文件，所以我建议你从那里引入`open`函数：






