
古老的 wc（字数统计）程序可以追溯到 AT&T UNIX 的第 1 版。该程序将显示 STDIN 或一个或多个文件中某些文本中找到的行数、字数和字节数

在本练习中，您将学到

* 如何使用 Iterator::all 函数
* 如何创建测试模块
* 如何伪造文件句柄进行测试
* 如何有条件地格式化和打印值
* 如何将一行文本分解为单词和字符
* 如何使用 Iterator::collect 将迭代器变成集合

## How wc Works

以下是 BSD wc 手册页的摘录，描述了该工具的工作原理


一张图片胜过千字节的文字，所以我将向您展示一些使用 05_wcr 目录中的测试文件的示例。
首先，考虑一个应该报告 0 行、字和字节的空文件，每个文件都应该在 8 个字符宽的字段中右对齐：
```sh
$ cd 05_wcr
$ wc tests/inputs/empty.txt
       0       0       0 tests/inputs/empty.txt
```

接下来，尝试一个只有一行文本的文件。
请注意，我在一些单词和制表符之间放置了不同数量的空格，原因稍后将讨论。
我将使用带有标志 -t 的 cat 将制表符显示为 ^I，并使用 -e 来显示行尾的 $：

```sh
$ cat -te tests/inputs/fox.txt
The  quick brown fox^Ijumps over   the lazy dog.$
```

这个例子足够短，我可以手动计算所有行、单词、字节，如图 5-1 所示，其中空格用凸起的点表示，制表符用 \t 表示，行尾用 $ 表示

我发现wc的意见是一致的

```sh
$ wc tests/inputs/fox.txt
       1       9      48 tests/inputs/fox.txt
```

如第 3 章中提到的，字节可能等同于 ASCII 字符，但 Unicode 字符可以占用多个字节。
文件tests/inputs/atlamal.txt包含来自Atlamál hin groenlenzku或古挪威诗歌《Atli的格陵兰民谣》的第一节

```sh
$ cat tests/inputs/atlamal.txt
Frétt hefir öld óvu, þá er endr of gerðu
seggir samkundu, sú var nýt fæstum,
æxtu einmæli, yggr var þeim síðan
ok it sama sonum Gjúka, er váru sannráðnir.
```

根据 wc，该文件包含 4 行、29 个字、177 个字节
```sh
$ wc tests/inputs/atlamal.txt
       4      29     177 tests/inputs/atlamal.txt
```

如果我只想要行数，我可以使用 -l 标志，并且只会显示该列

```sh
$ wc -l tests/inputs/atlamal.txt
       4 tests/inputs/atlamal.txt
```

我可以类似地仅通过 -c 请求字节数或通过 -w 请求单词数，并且仅显示这两列

```sh
$ wc -w -c tests/inputs/atlamal.txt
      29     177 tests/inputs/atlamal.txt
```

我可以使用 -m 标志请求字符数

```sh
$ wc -m tests/inputs/atlamal.txt
     159 tests/inputs/atlamal.txt
```

如果您同时提供标志 -m 和 -c，则 wc 的 GNU 版本将显示字符和字节计数，但 BSD 版本将仅显示其中一个，后一个标志优先：

```sh
$ wc -cm tests/inputs/atlamal.txt 
     159 tests/inputs/atlamal.txt
$ wc -mc tests/inputs/atlamal.txt 
     177 tests/inputs/atlamal.txt
```

请注意，无论 -wc 或 -cw 等标志的顺序如何，输出列始终按行、字和字节/字符排序

```sh
$ wc -cw tests/inputs/atlamal.txt
      29     177 tests/inputs/atlamal.txt
```

如果没有提供位置参数，wc 将从 STDIN 读取并且不会打印文件名：

```sh
cat tests/inputs/atlamal.txt | wc -lc
       4     173
```

wc 的 GNU 版本将理解文件名 - 表示 STDIN，并且它还提供长标志名称以及一些其他选项



如果处理多个文件，两个版本都将以一总行结束，显示所有输入的行数、字数和字节数

```sh
$ wc tests/inputs/*.txt
       4      29     177 tests/inputs/atlamal.txt
       0       0       0 tests/inputs/empty.txt
       1       9      48 tests/inputs/fox.txt
       5      38     225 total
```

在处理文件时，会注意到不存在的文件并向 STDERR 发出警告

```sh
$ wc tests/inputs/fox.txt blargh tests/inputs/atlamal.txt
       1       9      48 tests/inputs/fox.txt
wc: blargh: open: No such file or directory
       4      29     177 tests/inputs/atlamal.txt
       5      38     225 total
```

我还可以在 bash 中重定向文件句柄 2，以验证 wc 是否将警告打印到 STDERR

```sh
$ wc tests/inputs/fox.txt blargh tests/inputs/atlamal.txt 2>err 
       1       9      48 tests/inputs/fox.txt
       4      29     177 tests/inputs/atlamal.txt
       5      38     225 total
$ cat err 
wc: blargh: open: No such file or directory
```


述行为与挑战计划预期实施的行为一样多。有一个广泛的测试套件可以一路为您提供帮助

## Getting Started 开始


我们的 Rust 版本的 wc 挑战计划应该被称为 wcr（发音为 wick-er）。使用`cargo new wcr`启动，然后修改`Cargo.toml`以添加以下依赖项
```toml
[dependencies]
clap = "2.33"

[dev-dependencies]
assert_cmd = "1"
predicates = "1"
rand = "0.8"
```

将我的 05_wcr/tests 目录复制到您的新项目中并运行 Cargo test 以执行初始构建并运行测试，所有这些都应该失败。有时我可能是一个相当缺乏想象力的人，所以我将继续对 src/main.rs 使用我在之前的程序中使用的相同结构：

```rust
fn main() {

	if let Err(e) = wcr::get_args().and_then(wcr::run) {
	
	eprintln!("{}", e);
	
	std::process::exit(1);
	
	}

}
```

以下是 src/lib.rs 的框架，您可以复制。首先，我将如何定义 Config 来表示命令行参数：

```rust
use clap::{App, Arg};
use std::error::Error;

type MyResult<T> = Result<T, Box<dyn Error>>;

#[derive(Debug)]
pub struct Config {
  files: Vec<String>,
  lines: bool,
  words: bool,
  bytes: bool,
  chars: bool,
}
```

这是您开始使用时需要的两个功能。我会让你填写这个骨架的 get_args：

```rust
pub fn get_args() -> MyResult<Config> {
  let matches = App::new("wcr").version("0.1.0").author("saber").about("这是一个wc的复制版本").get_matches();
  Ok(Config { files: vec![], lines: false, words: false, bytes: false, chars: false })
}
```

我建议您通过打印配置来开始运行:

```rust
pub fn run(config: Config) -> MyResult<()> {
  println!("config: {:?}", config);
  Ok(())
}
```

尝试让您的程序生成如下所示的 --help 输出：

```sh
$ cargo run -- --help
wcr 0.1.0
Ken Youens-Clark <kyclark@gmail.com>
Rust wc

USAGE:
    wcr [FLAGS] [FILE]...

FLAGS:
    -c, --bytes      Show byte count
    -m, --chars      Show character count
    -h, --help       Prints help information
    -l, --lines      Show line count
    -V, --version    Prints version information
    -w, --words      Show word count

ARGS:
    <FILE>...    Input file(s) [default: -]
```

我有点担心是否要模仿 BSD 或 GNU 版本的 wc 来组合 -m（字符）和 -c（字节）标志。我决定使用 BSD 行为，因此您的程序应该禁止同时使用这两个标志：

```sh
 cargo run -- -cm tests/inputs/fox.txt
error: The argument '--bytes' cannot be used with '--chars'

USAGE:
    wcr --bytes --chars
```

默认行为是打印行、单词和字节，这意味着当用户没有明确请求时，配置中的这些值应该为 true。确保您的程序将打印此内容：

```sh
$ cargo run -- tests/inputs/fox.txt
Config {
    files: [ 
        "tests/inputs/fox.txt",
    ],
    lines: true,
    words: true,
    bytes: true,
    chars: false, 
}
```

如果存在任何一个标志，那么所有其他未提及的标志都应该是假的

```sh
$ cargo run -- -l tests/inputs/*.txt 
Config {
    files: [
        "tests/inputs/atlamal.txt",
        "tests/inputs/empty.txt",
        "tests/inputs/fox.txt",
    ],
    lines: true, 
    words: false,
    bytes: false,
    chars: false,
}
```

停在这里并开始工作。我的狗需要洗澡，所以我马上回来。

```rust
pub fn get_args() -> MyResult<Config> {
    let matches = App::new("wcr")
        .version("0.1.0")
        .author("saber")
        .about("这是一个wc的复制版本")
        .arg(
            Arg::with_name("files")
                .value_name("FILE")
                .help("输入文件")
                .default_value("-")
                .min_values(1),
        )
        .arg(
            Arg::with_name("lines")
                .value_name("LINES")
                .help("显示行数")
                .takes_value(false)
                .short("l")
                .long("lines"),
        )
        .arg(
            Arg::with_name("words")
                .value_name("WORDS")
                .help("显示单词数")
                .takes_value(false)
                .short("w")
                .long("words"),
        )
        .arg(
            Arg::with_name("bytes")
                .value_name("BYTES")
                .help("显示字节数")
                .takes_value(false)
                .short("c")
                .long("bytes"),
        )
        .arg(
            Arg::with_name("chars")
                .value_name("CHARS")
                .help("显示字符数")
                .takes_value(false)
                .short("m")
                .long("chars")
                .conflicts_with("bytes"),
        )
        .get_matches();
    let mut lines = matches.is_present("lines");
    let mut words = matches.is_present("words");
    let mut bytes = matches.is_present("bytes");
    let mut chars = matches.is_present("chars");
    // if lines == false && words == false && bytes == false && chars == false  {
    //   println!("lines is false");
    //   lines = true;
    //   words = true;
    //   bytes = true;
    //   chars = false;
    // }
    if [lines, words, bytes, chars].iter().all(|v| v == &false) {
        lines = true;
        words = true;
        bytes = true;
        chars = false;
    }
    Ok(Config {
        files: matches.values_of_lossy("files").unwrap(),
        lines,
        words,
        bytes,
        chars,
    })
}
```

我想强调的是，我创建了一个包含所有标志的切片，并调用 slice::iter 方法来创建迭代器。
这样我就可以使用 Iterator::all 函数来查找是否所有值都为 false。
此方法需要一个闭包，它是一个匿名函数，可以作为参数传递给另一个函数。
在这里，闭包是一个谓词或一个测试，用于确定元素是否为假。
我想知道是否所有标志都为 false，因此对每个标志的引用都会作为闭包的参数传递。
这些值是引用，因此我必须将每个值与 &false 进行比较，后者是对布尔值的引用。
如果所有评估都为 true，则 Iterator::all 将返回 true

> 采用闭包的迭代器方法
> 您应该花一些时间阅读 Iterator 文档，以注意使用闭包作为参数来选择、测试或转换元素的其他方法，包括
> * 如果某项闭包的一次评估返回 true，则 Iterator::any 将返回 true。
> * Iterator::filter 将查找谓词为 true 的所有元素。
> * Iterator::map 将对每个元素应用闭包，并返回带有转换后的元素的 std::iter::Map
> * Iterator::find 将返回满足谓词 Some(value) 的迭代器的第一个元素，如果所有元素的计算结果均为 false，则返回 None。
> * Iterator::position 将返回满足谓词的第一个元素的索引，如 Some(value) 或 None（如果所有元素计算结果为 false）。
> * Iterator::cmp、Iterator::min_by 和 Iterator::max_by 具有接受项目对进行比较的谓词，以分别找到最小值和最大值。

## 迭代文件

现在开始计数。您可以从处理每个文件、计算各个位并打印所需的列开始。我建议您再次使用第 2 章中的 open 函数来打开文件：

```rust
fn open(filename: &str) -> MyResult<Box<dyn BufRead>> {
  match filename {
      "-" => Ok(Box::new(BufReader::new(io::stdin()))),
      _ => Ok(Box::new(BufReader::new(File::open(filename)?)))
  }
}
```

```rust
use clap::{App, Arg};
use std::error::Error;
use std::fs::File;
use std::io::{self, BufRead, BufReader};
```

```rust
pub fn run(config: Config) -> MyResult<()> {
    println!("config: {:?}", config);
    for filename in &config.files  {
        match open(filename) {
            Err(err) => eprintln!("{}: {}", filename, err),
            Ok(_file) => println!("Opened: {}", filename)
        }
    }
    Ok(())
}
```

使用打开的文件句柄，尽可能简单地使用空文件开始，并确保您的程序为行、字和字节三列打印零：

```sh 
$ cargo run -- tests/inputs/empty.txt
       0       0       0 tests/inputs/empty.txt
```

接下来使用tests/inputs/fox.txt并确保获得以下计数。我特地添加了各种类型和数量的空格来挑战你如何将文本拆分为单词:

```sh
$ cargo run -- tests/inputs/fox.txt
       1       9      48 tests/inputs/fox.txt
```

确保你的程序可以正确处理tests/inputs/atlamal.txt中的Unicode

```sh
$ cargo run -- tests/inputs/atlamal.txt
       4      29     177 tests/inputs/atlamal.txt
```

并且你正确地计算了字符：

```sh
$ cargo run -- tests/inputs/atlamal.txt -wml
       4      29     159 tests/inputs/atlamal.txt

```

当您可以处理任何一个文件时，请使用多个文件来检查是否打印了正确的总列：

```sh
$ cargo run -- tests/inputs/*.txt
       4      29     177 tests/inputs/atlamal.txt
       0       0       0 tests/inputs/empty.txt
       1       9      48 tests/inputs/fox.txt
       5      38     225 total
```

当一切正常后，尝试从 STDIN 读取

```sh
$ cat tests/inputs/atlamal.txt | cargo run
       4      29     177
```

经常进行`cargo test`，看看进展如何。在通过所有测试之前，请不要继续阅读。

## 解决方案

我想向您介绍我是如何得出解决方案的，并且我想强调的是，我的方法只是编写此程序的多种可能方法之一。只要您的代码通过测试并产生与 BSD 版本的 wc 相同的输出，那么它就可以正常工作，您应该为自己的成就感到自豪。

### 计算文件或 STDIN 的元素

我建议您首先迭代文件名并打印文件的计数。为此，我决定创建一个名为 count 的函数，它接受一个文件句柄，并可能返回一个名为 FileInfo 的结构，其中包含行数、单词数、字节数和字符数，每个都表示为 usize。我说该函数可能会返回这个结构体，因为该函数将涉及 IO，这可能会出现偏差。我将以下定义放在 Config 结构之后。出于我稍后将解释的原因，除了 Debug 之外，还必须派生出 PartialEq 特征：

```rust
#[derive(Debug, PartialEq)]
pub struct  FileInfo {
  num_lines: usize,
  num_words: usize,
  num_bytes: usize,
  num_chars: usize,
}
```

为了表示该函数可能成功或失败，它将返回一个 `MyResult<FileInfo>`，这意味着成功时它将返回 `Ok<FileInfo>`，失败时它将返回 Err。为了启动这个函数，我将初始化一些可变变量来计算所有元素并返回一个 FileInfo 结构：

``` rust 
pub fn count(mut file: impl BufRead) -> MyResult<FileInfo> {
  Ok(FileInfo { num_lines: 0, num_words: 0, num_bytes: 0, num_chars: 0 })
}
```

> 我引入 impl 关键字来指示文件值必须实现 BufRead 特征。回想一下，open 返回一个满足此条件的值。您很快就会看到这如何使该功能变得灵活。


在第 3 章中，我展示了如何编写单元测试，将其放在正在测试的函数之后。我将为 count 函数创建一个单元测试，但这一次我将把它放在一个名为测试的模块中。这是一种对单元测试进行分组的简洁方法，我可以使用一个配置选项来告诉 Rust 仅在测试期间编译模块。
这特别有用，因为我想在测试中使用 std::io::Cursor 为 count 函数创建一个假文件句柄。
当我在不测试的情况下构建和运行程序时，不会包含该模块。

“Cursor”与内存缓冲区（任何实现 `AsRef<[u8]>` 的东西一起使用），以允许它们实现读取和/或写入，允许这些缓冲区在您可能使用执行实际操作的读取器或写入器的任何地方使用。 /O。”
以下是我如何创建测试模块，然后导入并测试计数函数

使用`cargo test test_count` 运行此测试。您将看到 Rust 编译器发出的许多关于未使用的变量或不需要可变的变量的警告。最重要的结果是测试失败：

```sh
running 1 test
test tests::test_count ... FAILED

failures:

---- tests::test_count stdout ----
thread 'tests::test_count' panicked at 'assertion failed: `(left == right)`
  left: `FileInfo { num_lines: 0, num_words: 0, num_bytes: 0, num_chars: 0 }`,
 right: `FileInfo { num_lines: 1, num_words: 10, num_bytes: 48, num_chars: 48 }`', src/lib.rs:124:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::test_count

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

花一些时间编写将通过此测试的计数函数的其余部分。我会带着我那干干净净的狗去散步，也许还会喝点茶。

好的，我们回来了。现在我将向您展示我是如何编写计数函数的。
我从第 3 章知道 BufRead::lines 会删除行结尾，但我不希望这样做，因为 Windows 文件中的换行符是两个字节 (\r\n)，但 Unix 换行符只是一个字节 (\n)。我可以复制第 3 章中的一些代码，使用 BufRead::read_line 将每一行读入缓冲区。方便的是，这个函数告诉我已经从文件中读取了多少字节：
