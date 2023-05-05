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



