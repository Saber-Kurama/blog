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



