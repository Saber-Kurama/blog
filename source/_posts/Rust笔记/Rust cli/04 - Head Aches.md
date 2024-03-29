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
```rust
fn open(filename: &str) -> MyResult<Box<dyn BufRead>> {
    match filename {
        "_" => Ok(Box::new(BufReader::new(io::stdin()))),
        _ => Ok(Box::new(BufReader::new(File::open(filename)?))),
    }
}
```
请务必添加所有需要的依赖项：

```rust
use std::{
    error::Error,
    format,
    fs::File,
    io::{self, BufRead, BufReader},
};

use clap::{App, Arg};
```

展开您的运行功能，尝试打开文件，在遇到错误时打印错误：

```rust
 for filename in config.files {
        match open(&filename) {
            Err(err) => eprintln!("{} : {}", filename, err),
            Ok(_file) => println!("Opend {}", filename),
        }
    }
```

使用好文件和坏文件运行您的程序，以确保它似乎工作：

```rust
$ cargo run -- blargh tests/inputs/one.txt
blargh: No such file or directory (os error 2)
Opened tests/inputs/one.txt
```

接下来，尝试求解读取给定文件的行和字节，然后尝试添加分隔多个文件参数的标头。在处理无效文件时，请仔细查看头部的错误输出。请注意，可读文件首先有一个标题，然后是文件输出，但无效文件只会打印错误。此外，还有一条额外的空白行将有效文件的输出分开：

```shell
❯ head -n 1 tests/inputs/one.txt blargh tests/inputs/two.txt
==> tests/inputs/one.txt <==
head: blargh: No such file or directory
One lines, four words.
==> tests/inputs/two.txt <==
Two lines,
```

我专门设计了一些具有挑战性的输入，供您考虑。要查看您面临的内容，请使用文件命令报告文件类型信息：


> 在Windows上，换行符是回车和换行的组合，通常显示为CRLF或\r\n。在Unix平台上，只使用换行，所以LF或\n。这些行结尾必须保留在程序的输出中，因此您必须找到一种在不删除行结尾的情况下读取文件中行的方法。

### 阅读字节与字符

我想解释一下从文件中读取字节和字符之间的区别。20世纪60年代初，美国信息交换标准代码（ASCII，发音为键）128个字符的表格代表了计算中所有可能的文本元素。只需要七位（27 = 128）来表示每个字符，因此字节和字符的概念是可以互换的。

由于创建了Unicode（通用编码字符集）来表示世界上所有的书写系统（甚至表情符号），一些字符可能需要最多四个字节。Unicode标准定义了几种编码字符的方法，包括UTF-8（使用8位的Unicode转换格式）。正如我所指出的，文件tests/inputs/one.txt以字符Ő开头，在UTF-8中为两个字节长。如果你想向你展示这个字符，你必须请求两个字节：


如果我让Head从这个文件中只选择第一个字节，我会得到字节值195，这不是有效的UTF-8字符串。输出是一个特殊字符，表示将字符转换为Unicode时存在问题


挑战计划预计将重现这种行为。

这是一个具有挑战性的编写程序，但您应该能够使用std::io、std::fs::File和std::io::BufReader来了解如何读取每个文件中的字节和行。我在tests/cli.rs中包含了一整套测试，您应该将其复制到源树中。请务必经常运行`cargo test`，以检查您的进度。在查看我的解决方案之前，请尽最大努力通过所有测试。

## 解决

我真的很惊讶我通过写这个程序学到了很多东西。事实证明，我期望的一个相当简单的程序非常具有挑战性。我想向您介绍我是如何找到解决方案的，从我如何逐行阅读文件开始。

### 逐行读取文件

首先，我将修改第3章中的一些代码，用于读取文件中的行：
```rust
 match open(&filename) {
            Err(err) => eprintln!("{} : {}", filename, err),
            Ok(file) => {
                for line in file.lines().take(config.lines) {
                    println!("{}", line?)
                }
            }
        }
```

我认为这是一个非常有趣的解决方案，因为它使用Iterator::take方法从config.lines中选择行数。
我可以运行该程序，从包含三行的文件中选择一行，它似乎工作得很棒：

如果我运行`cargo test`，该程序将通过几次测试，这似乎只实现了一小部分规格。它从三个使用Windows编码的输入文件开始的测试都失败了。为了解决这个问题，我要坦白一下。

### 读取文件时保留行结尾

亲爱的读者，我很痛苦地告诉你这些，但我在第3章中骗了你。我展示的catr程序没有完全复制原始程序，因为它使用`BufRead::lines`来读取输入文件。该函数的文档显示“返回的每个字符串在末尾不会有一个换行符字节（0xA字节）或CRLF（0xD，0xA字节）。”
我希望你能原谅我，因为我想向你展示阅读文件的行有多容易，但你应该知道，catr程序用Unix风格的换行符取代了Windows CRLF行结尾。

要解决这个问题，我必须使用`BufRead::read_line`，上面写着“此函数将从底层流中读取字节，直到找到换行符（0xA字节）或EOF1。一旦找到，直到并包括分隔符（如果找到）的所有字节都将附加到buf。”以下是将保留原始行结尾的版本。
有了这些变化，该程序将通过比失败更多的测试：

```rust
 for filename in config.files {
        match open(&filename) {
            Err(err) => eprintln!("{} : {}", filename, err),
            Ok(mut file) => {
                let mut line = String::new();
                for _ in 0..config.lines {
                    let bytes = file.read_line(&mut line)?;
                    if bytes == 0 {
                        break;
                    }
                    println!("{}", line);
                    line.clear();
                }
            }
        }
    }
```

1. 接受文件处理为mut（可变）值。
2. 使用`String::new`创建一个新的空可变字符串缓冲区来保存每行。
3. 用于迭代std::ops::Range，从0计数到请求的行数。变量名_表示我不打算使用它
4. 使用`BufRead::read_lin`e阅读下一行。
5. 文件处理到达终点时将返回0字节，因此请突破循环。
6. 打印行，包括原始行尾。
7.  使用`String::clea`r清空行缓冲区。

如果我在这一点上运行`cargo test`，我几乎通过了所有读取行的测试，而读取字节和处理多个文件的所有测试都失败了。

### 从文件中读取字节

接下来，我将处理从文件中读取字节。在我尝试打开文件后，我会检查config.bytes是否是一些字节数；否则，我将使用前面读取行的代码：

```rust
  if let Some(num_bytes) = config.bytes {
                    let mut handle = file.take(num_bytes as u64);
                    let mut buffer = vec![0; num_bytes];
                    let n = handle.read(&mut buffer)?;
                    println!("{}", String::from_utf8_lossy(&buffer[..n]));
                }
```

1. 使用模式匹配来检查config.bytes是否是要读取的字节数。
2. 使用`take`读取请求的字节数。
3. 创建一个固定长度的num_bytes的可变缓冲区，填充零，以保存从文件中读取的字节。
4. 从文件处理中读取所需的字节数到缓冲区。值n将报告实际读取的字节数，该字节数可能小于请求的数量。
5. 将字节转换为可能无效的UTF-8的字符串。注意范围操作，仅选择实际读取的字节。

> 来自std::io::Read trait的take方法希望其参数为u64类型，但我有一个usize。我使用as关键字转换或转换值。

对我来说，这可能是该计划中最难的部分。一旦我弄清楚如何只读几个字节，我就必须弄清楚如何将它们转换为文本。如果我只使用多字节字符的一部分，结果将失败，因为Rust中的字符串必须是有效的UTF-8。我很高兴找到String::from_utf8_lossy，它将悄悄地将无效的UTF-8序列转换为未知或替换字符：

```rust
❯ cargo run --  -c 1  tests/inputs/one.txt
   Compiling headr v0.1.0 (/Users/saber/coding/rust/command-line-rust/04_headr/headr)
    Finished dev [unoptimized + debuginfo] target(s) in 1.17s
     Running `target/debug/headr -c 1 tests/inputs/one.txt`
�
```

让我向您展示我尝试从文件中读取字节的第一个方法。我决定将整个文件读成一个字符串，将其转换为字节向量，并使用切片来选择第一个`num_bytes。`

```rust
	let mut contents = String::new();
	file.read_to_string(&mut contents)?;
	let bytes = contents.as_bytes();
	println!("{}", String::from_utf8_lossy(&bytes[..num_bytes]));
```

1. 创建一个新的字符串缓冲区来保存文件的内容。
2. 将整个文件内容读入字符串缓冲区。
3. 使用str::as_bytes将内容转换为字节（u8或无符号8位整数）。
4. 使用String::from_utf8_lossy将字节的切片转换为字符串。

> 我向您展示这种方法，以便您知道如何将文件读取到字符串中；但是，如果文件大小超过您机器上的内存量，这可能是一件非常危险的事情。一般来说，除非你确定文件很小，否则这是一个糟糕的想法。


前面代码的另一个严重问题是，它假设切片操作字节[..num_bytes]将成功。例如，如果您将此代码与空文件一起使用，您将要求字节不存在。这将导致您的程序崩溃并立即退出，并显示错误消息：

```shell
❯ cargo run --  -c 1  tests/inputs/empty.txt
thread 'main' panicked at 'range end index 1 out of range for slice of length 0', src/lib.rs:79:61
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

> Rust可以防止你犯各种令人震惊的错误，但它不能阻止你做蠢事。你仍然有很多方法可以射中自己的脚。

以下可能是从文件中读取所需字节数的最短方法：

```rust
let bytes: Result<Vec<_>, _> = file.bytes().take(num_bytes).collect();
print!("{}", String::from_utf8_lossy(&bytes?))
```

在前面的代码中，类型注释`Result<Vec<_>，_>`是必要的，因为编译器将字节的类型推断为切片，其大小未知。我必须表明我想要一个Vec，这是一个堆分配内存的智能指针。这里的下划线`（_）`表示部分类型注释，这基本上指示编译器推断类型。
没有这个，编译器会这样抱怨：

```shell
Compiling headr v0.1.0 (/Users/kyclark/work/sysprog-rust/playground/headr)
error[E0277]: the size for values of type `[u8]` cannot be known at compilation time
  --> src/lib.rs:95:58
   |
95 |                     print!("{}", String::from_utf8_lossy(&bytes?));
   |                                                          ^^^^^^^ doesn't
   |                                        have a size known at compile-time
   |
   = help: the trait `Sized` is not implemented for `[u8]`
   = note: all local variables must have a statically known size
   = help: unsized locals are gated as an unstable feature
```

> 您现在已经看到，下划线_具有各种不同的功能。作为变量的前缀或名称，它显示您不想使用该值的编译器。在match中，它是处理任何案件的通配符。当用于类型注释时，它会告诉编译器推断类型。

您还可以使用`turbofish`运算符`（::<>）`在表达式右侧指示类型信息。通常，无论您在左侧还是右侧指示类型，这都是一个风格问题，但稍后您将看到一些表达式需要涡轮鱼的例子：

```rust
let bytes = file.bytes().take(num_bytes).collect::<Result<Vec<_>, _>>();
```


`String::from_utf8_lossy（b'\xef\xbf\xbd'）`产生的未知字符与BSD头（b'\xc3'）产生的输出不完全相同，这使得这有点难以测试。
如果您查看tests/cli.rs中的运行帮助函数，您会发现我读取了期望值（来自头部的输出），并使用相同的函数来转换可能无效的UTF-8，以便我可以比较两个输出。
Run_stdin函数的工作原理类似：

1. 在expected_file中处理任何无效的UTF-8。
2. 将输出和预期值比较为字节片（[u8]）。



### 打印文件分隔符

最后要处理的是多个文件之间的分隔符。如前所述，有效文件有一个标头，将文件名放在`==>和<==`标记中。第一个文件之后的文件在开头有一个额外的换行，以直观地分隔输出。这意味着我需要知道我正在处理的文件的编号，我可以通过使用`Iterator::enumerate`方法获得。
以下是我的运行函数的最终版本，它将通过所有测试：

```rust
pub fn run(config: Config) -> MyResult<()> {
    let num_files = config.files.len();
    for (file_num, filename) in config.files.iter().enumerate() {
        match open(&filename) {
            Err(err) => eprintln!("{} : {}", filename, err),
            Ok(mut file) => {
                if num_files > 1 {
                    println!(
                        "{}===> {} <===",
                        if file_num > 0 { "\n" } else { "" },
                        &filename
                    )
                }
                if let Some(num_bytes) = config.bytes {
                    let mut handle = file.take(num_bytes as u64);
                    let mut buffer = vec![0; num_bytes];
                    let n = handle.read(&mut buffer)?;
                    println!("{}", String::from_utf8_lossy(&buffer[..n]));
                } else {
                    let mut line = String::new();
                    for _ in 0..config.lines {
                        let bytes = file.read_line(&mut line)?;
                        if bytes == 0 {
                            break;
                        }
                        println!("{}", line);
                        line.clear();
                    }
                }
            }
        }
    }
    Ok(())
}
```


1. 使用Vec::len方法获取文件数量。
2. 使用Iterator::enumerate方法跟踪文件号和文件名。
3. 仅在有多个文件时打印标题。当file_num大于0时打印换行，这表示第一个文件。



## 更进一步

* 实现GNU版本的乘数后缀，例如，-c=1K意味着打印文件的前1024字节。请务必添加并运行测试。
* 实现GNU版本的负数选项，其中-n=-3意味着您应该打印文件的最后三行以外的所有内容。一如既往，创建测试以确保您的程序正确无误。
* 添加一个选择字符的选项。
* 将带有Windows行结尾的文件添加到第3章的测试中。编辑该程序的mk-outs.sh以合并此文件，然后展开测试和程序，以确保保留行尾。



## 总结

本章深入探讨了一些相当棘手的主题，例如将&str转换为usize，将字符串转换为错误，以及将usize转换为u64等类型。

我觉得我花了很长时间才理解&str和String之间的区别，以及为什么我需要使用From::from来创建MyResult的Err部分。

如果你仍然感到困惑，只要知道你不会一直困惑。

我认为，如果你继续阅读文档并编写更多代码，它最终会来到你身边。

以下是你在这个练习中完成的一些事情：

* 您学会了创建可以获取值的可选参数。以前，选项是旗帜。
* 您看到所有命令行参数都是字符串。您使用str::parse方法尝试将像“3”这样的字符串转换为数字3。
* 您学会了如何为单个函数编写和运行单元测试。
* 您学会了使用as关键字或从和Into等特征转换类型。
* 您发现_作为值的名称或前缀是向编译器表明您不打算使用变量的一种方式。当用于类型注释时，它会告诉编译器推断类型。
* 你了解到，匹配臂可以包含一个额外的布尔条件，称为后卫。
* 您学会了如何在读取文件句柄时使用BufRead::read_line来保留行结尾。
* 您发现take方法适用于迭代器和文件处理，以限制您选择的元素数量。
* 您学会了使用涡轮鱼在作业的左侧或右侧指示类型信息。
























