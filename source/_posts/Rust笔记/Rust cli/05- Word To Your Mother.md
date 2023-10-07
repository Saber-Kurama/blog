
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
        files: vec![],
        lines,
        words,
        bytes,
        chars,
    })
}
```