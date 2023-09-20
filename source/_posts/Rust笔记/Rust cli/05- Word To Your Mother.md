
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