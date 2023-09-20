
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


