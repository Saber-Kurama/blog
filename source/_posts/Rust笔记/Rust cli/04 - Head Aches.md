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

## 开始

