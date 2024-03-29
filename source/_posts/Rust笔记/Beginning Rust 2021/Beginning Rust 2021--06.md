---
 title: Rust 2021 版入门
 date: 2022-01-18
 tag: rust
---

# RUST

## Data Implementation 数据实现
*  How to know how many bytes of stack are taken by objects of various types （如何知道各种类型的对象占用了多少字节的堆栈）
* How to shorten the path to access functions declared in external modules （如何缩短访问外部模块中声明的函数的路径）
* How bits are stored in primitive type objects （bits 如何存储在原始类型对象中）
* How to know where an object is stored in memory （如何知道对象在内存中的存储位置）
* Why padding can increase the size taken by some objects （为什么填充可以增加某些对象的大小）
* How vectors are implemented （向量是如何实现的）

### Discovering the Size of Objects （发现Objects的大小）

给定一个源文件，Rust 编译器可以自由生成任何机器代码，只要它以 Rust 语言为该源代码指定的方式运行
因此，例如，给定一个变量，它使用了多少内存位，以及它在内存中的位置都没有定义。编译器甚至可以从内存中删除该变量，因为它从未使用过，或者因为它保存在处理器寄存器中。
然而，看看 Rust 程序使用的数据排列的可能典型实现是有启发性的。
为此，可以使用一些 Rust 功能：
获取字节大小
```rust
print!("{} ", std::mem::size_of::<i32>()); // 4
print!("{} ", std::mem::size_of_val(&12)); // 4
```

### The use Directive 使用指令
如果必须多次指定到达库函数的路径，使用 use 指令将全部或部分路径导入当前范围很方便
前面的例子可以这样改写
``` rust
use std::mem::size_of;
use std::mem::size_of_val;
print!("{} ", size_of::<i32>());
print!("{} ", size_of_val(&12));
```

### The Sizes of the Primitive Types  (原始类型的大小)

``` rust
use std::mem::*;
print!("{} {} {} {} {} {} {} {} {} {} {} {} {} {}",
    size_of::<i8>(),
    size_of::<u8>(),
    size_of::<i16>(),
	size_of::<u16>(),
    size_of::<i32>(),
    size_of::<u32>(),
    size_of::<i64>(),
    size_of::<u64>(),
    size_of::<i128>(),
    size_of::<u128>(),
    size_of::<f32>(),
    size_of::<f64>(),
    size_of::<bool>(),
    size_of::<char>());
// In any computer, this will print: 1 1 2 2 4 4 8 8 16 16 4 8 1 4.
```

### The Representation of Primitive Types (原始类型的表示)

Rust 不鼓励访问对象的内部表示，所以这并不容易；但有一个技巧可以做到这一点：

``` rust
fn as_bytes<T>(o: &T) -> &[u8] {
    unsafe {
        std::slice::from_raw_parts(
            o as *const _ as *const u8,
            std::mem::size_of::<T>())
    }
}
println!("{:?}", as_bytes(&1i8));
println!("{:?}", as_bytes(&2i16));
println!("{:?}", as_bytes(&3i32));
println!("{:?}", as_bytes(&(4i64 + 5 * 256 + 6 * 256 * 256)));
println!("{:?}", as_bytes(&'A'));
println!("{:?}", as_bytes(&true));
println!("{:?}", as_bytes(&&1i8));
```
In an x86_64 system, this could print:
```
[1]
[2, 0]
[3, 0, 0, 0]
[4, 5, 6, 0, 0, 0, 0, 0]
[65, 0, 0, 0]
[1]
[129, 165, 54, 102, 23, 86, 0, 0]
```
通用函数` as_bytes` 使用了一些我们还没有见过的 Rust 结构，这里不会解释，因为理解它的作用不需要他们的知识。它只是简单地引用任何类型的参数，并返回一个对象，该对象表示该对象中包含的字节序列。通过打印这样的对象，您可以将任何对象的表示形式视为它存储在内存中的字节序列
首先，一个值为 1 的 i8 存储在一个字节中。这在任何受支持的硬件架构中都是一样的。
然后，将值为 2 的 i16 存储为一对字节，其中第一个为 2，第二个为 0。这在 32 位和 64 位处理器上都会发生，但仅在所谓的 little -endian 硬件体系结构，即存储多字节数字的硬件体系结构，将最低有效字节放在最低地址。相反，大端硬件架构会打印 [0, 2]。
请注意，char 存储为包含此类字符的 Unicode 值的 32 位数字，bool 存储为单个字节，1 表示真，0 表示假。
最后，最后一条语句打印一个 i8 数字的地址。对于 64 位处理器，这样的地址占用 8 个字节，并且每次运行都不同.

### Location of Bytes in Memory  内存中字节的位置

您还可以发现任何对象的（虚拟）内存位置，即它的地址：

``` rust
let b1 = true;
let b2 = true;
let b3 = false;
print!("{} {} {}",
    &b1 as *const bool as usize,
    &b2 as *const bool as usize,
    &b3 as *const bool as usize);
```

In a 64-bit system, this will print three huge numbers, resembling `140729279188109 ` `140729279188110`  `140729279188111`. 

但是，有一种更简单的方法可以以十六进制表示法打印任何对象的地址:

```rust
let b1 = true;
let b2 = true;
let b3 = false;
print!("{:p} {:p} {:p}", &b1, &b2, &b3);
```
It will print something like:
`0x7ffe16b20c8d` `0x7ffe16b20c8e` `0x7ffe16b20c8f`

这是前面代码中三个对象位置的表示。
| Absolute Address | Binary Value | Variable Name |Type|
| --- | --- | --- | --- | 
| 140729279188109 | 0000_0001 | b1 | bool |
| 140729279188110 | 0000_0001 | b2 | bool |
| 140729279188111 | 0000_000  | b3 | bool |

三个对象中的每一个都只占用一个字节。第一个打印的数字是 b1 变量的地址；第二个是b2变量的地址；第三个是 b3 变量的地址。看起来，这三个数字是连续的，这意味着这三个对象被分配在连续的虚拟内存位置。
第一个数字包含真正的布尔值，它由 1 个字节表示，该字节又由 7 个具有零值的位和一个具有一个值的位组成。第二个对象也包含真实值。相反，第三个对象包含错误值，由具有零值的八位表示。

在分配单个字节时，Rust 编译器通常按顺序排列它们，但在分配更大的对象时，它们在内存中的位置不容易预测。

几乎所有现代处理器都要求基本数据具有特定的内存位置，因此 Rust 放置其对象以便处理器可以轻松访问它们。

典型的对齐规则是这样的：“基本类型的每个对象都必须有一个地址是其自身大小的倍数。

因此，虽然只占用一个字节的对象可以放在任何地方，但占用两个字节的对象只能放在偶数地址；占用四个字节的对象只能放置在能被四整除的地址上；占用 8 个字节的对象只能放置在 8 的倍数的地址上。

此外，较大的对象的地址通常是 16 的倍数。

因此，这种对齐要求会产生未使用的空间，即所谓的填充。

### Sizes of Composite Data Types  复合数据类型的大小

当有一系列复合对象时出现填充的效果
``` rust
enum E1 { E1a, E1b }
enum E2 { E2a, E2b(f64) }
use std::mem::*;
print!("{} {} {} {} {} {}",
    size_of_val(&[0i16; 80]), // 80 * 2 = 160
    size_of_val(&(0i16, 0i64)), // 16 字节 ，6个字节填充
    size_of_val(&[(0i16, 0i64); 100]),  // 16 * 100 = 1600 
    size_of_val(&E1::E1a), // 一个没有数据字段的所有变体的枚举只占用一个字节；只要不超过 256 个变体，单个字节就可以包含一个标识变体的值；这种隐藏的值通常被命名为标签。
    size_of_val(&E2::E2a), // 最大变体包含 8 字节数字的枚举占用 16 字节，即使当前值没有数据；这是因为变量的任何值都必须占据相同的大小，即该枚举的最大可能值的大小。在这种情况下，第二个变体有一个 1 字节的标签和一个 8 字节的数字字段；该字段的地址必须是 8 的倍数，因此在标签和字段之间添加了一个 7 字节的填充。
    size_of_val(&vec![(0i16, 0i64); 100])); // 24 字节
```
This will print: 160 16 1600 1 16 24.

让我们看看向量的情况。
放入堆栈的数据必须在编译时具有已知的大小，因此数组可以在堆栈中完全分配；而对于向量，只有一个固定大小的头可以放在堆栈中，而剩余的数据必须在堆中分配。

### Vector Allocation  向量分配

我们看到向量必须实现为两个对象结构：堆栈分配的固定大小的标头和堆分配的可变长度缓冲区。
理论上有几种可能的方式来实现向量数据结构。
一种方法是在标头中只保留一个指向缓冲区的指针。
这样做的缺点是，每次需要数组的长度时，都需要一个间接级别。数组的长度经常被隐式或显式地需要，因此最好将这些信息保留在标头中。
实现缓冲区的一种天真的方法是将其大小调整到足以保留所需数据的大小。例如，如果请求包含 9 个 i32 项的向量，则将在堆中分配一个 9 * 4 字节的缓冲区。
只要向量不增长，这很好。但是如果另一个项被推入向量中，则必须重新分配缓冲区，并且堆分配和释放是昂贵的。此外，必须将旧缓冲区内容复制到新缓冲区。
如果构造一个包含 1,000 项的向量，一次一项，通过创建一个空向量并调用 push 函数 1,000 次，将有 1,000 次堆分配，999 次堆释放，以及 1000 * 999 / 2 == 499_500 个副本物品。
为了改善这种糟糕的性能，可能会分配更大的缓冲区，这样只有在缓冲区不够时才会执行重新分配。
因此，需要跟踪分配缓冲区中的位置数量和该缓冲区中已使用位置的数量。
分配的缓冲区中的位置数通常称为容量，这也是用于访问该数的函数的名称。这是一些显示向量缓冲区大小的代码： 

``` rust
let mut v = vec![0; 0];
println!("{} {}", v.len(), v.capacity());
v.push(11);
println!("{} {}", v.len(), v.capacity());
v.push(22);
println!("{} {}", v.len(), v.capacity());
v.push(33);
println!("{} {}", v.len(), v.capacity());
v.push(44);
println!("{} {}", v.len(), v.capacity());
v.push(55);
println!("{} {}", v.len(), v.capacity());
```
This will possibly print:
```
0 0
1 4
2 4
3 4
4 4
5 8
```
当一个空向量被创建时，它包含零个项目，它甚至没有分配一个堆缓冲区，所以它的容量也是零。
当添加第一项时，向量对象在堆中分配一个能够包含四个 32 位数字的缓冲区（即 16 字节缓冲区），因此它的容量为 4，但实际上它只包含一个项，所以它的长度是1。
当其他三个项目添加到向量中时，无需分配内存，因为预分配的缓冲区足够大以容纳它们。
但是当第五项添加到向量中时，必须分配更大的缓冲区。新缓冲区可容纳 8 个项目。
请注意，缓冲区的这种增长策略是一个实现细节，它可能会在标准库的未来实现中发生变化。
v对象在栈中存储了三个子对象：一个指向堆分配缓冲区的指针，它是一个内存地址；这种缓冲区的容量，作为一个项目的数量，它是一个使用大小；向量的长度作为项目的数量，这是一个小于或等于容量的使用数量。
因此，任何向量的头部在任何 64 位系统中占用 3 * 8 == 24 字节，在任何 32 位系统中占用 3 * 4 == 12 字节。

让我们看看如果我们将一千个项目添加到一个 32 位数字的向量中会发生什么:

``` rust
let mut v = vec![0; 0];
let mut prev_capacity = std::usize::MAX;
for i in 0..1_000 {
    let cap = v.capacity();
    if cap != prev_capacity {
        println!("{} {} {}", i, v.len(), cap);
        prev_capacity = cap;
    }
    v.push(1);
}
```

This could print:
```
0 0 0
1 1 4
5 5 8
9 9 16
17 17 32
33 33 64
65 65 128
129 129 256
257 257 512
513 513 1024
```
嗯，它也会为大多数其他类型的项目打印相同的向量。
变量 cap 存储向量的当前容量；变量 prev_capacity 存储了向量之前的容量，它被初始化为一个巨大的值。
在每次迭代中，在向向量添加项目之前，都会检查容量是否发生了变化。每次容量变化时，都会打印插入项目的数量和当前容量。
看起来容量始终是 2 的幂，并且所有 2 的幂都通过了，只是跳过了值 2。这样，只有 9 次分配，8 次释放，4 + 8 + 16 + 32 + 64 + 128 + 256 + 512 == 1020 份。因此，实际的标准库算法比本节开头描述的简单版本要高效得多。
一般来说，通过一次推送一个项目来填充一个向量会导致多次分配和释放，这是项目总数的对数，所以一个比项目数小得多的数字，对于一个相当大的数字物品。
因此，一次添加一个项目非常有效。但是，如果您可以估计将包含在向量中的项目数，则可以通过预先分配所需的缓冲区来提高效率，就像下面的代码一样：

``` rust
let mut v = Vec::with_capacity(800);
let mut prev_capacity = std::usize::MAX;
for i in 0..1_000 {
    let cap = v.capacity();
    if cap != prev_capacity {
        println!("{} {} {}", i, v.len(), cap);
        prev_capacity = cap;
    }
    v.push(1);
}
```

It will print:
```
0 0 800
801 801 1600
```

这个程序与前一个程序相同，除了第一行。函数 Vec::with_capacity 创建一个带有预分配缓冲区的向量，该缓冲区足够大以包含指定数量的项目。它指定了 800 个项目的容量，因此在超过该容量之前，初始化后不会发生分配。推到第801个时，容量就冲到了1600个。

## Defining Closures  定义闭包
* Why there is a need for anonymous inline functions, with type inference for the arguments and the return value type, without having to write braces, and which can access the variables that are alive at the function definition point（为什么需要匿名内联函数，对参数和返回值类型进行类型推断，无需编写大括号，并且可以访问在函数定义点处有效的变量）
* How such lightweight functions, named “closures,” can be declared and invoked（如何声明和调用这种名为“闭包”的轻量级函数）

###  The Need for Disposable Functions（对一次性功能的需求）

``` rust
```

### Capturing the Environment (捕捉环境)
本章到目前为止所说的一切在许多其他语言中也适用，包括 C 语言。但是 Rust 函数还有一个不寻常的限制：它们不能访问在它们之外声明的任何变量。您可以访问静态项，也可以访问 const 项，但不能访问堆栈分配（即“let-declared”）变量。例如，这个程序是非法的：

``` rust
let two = 2.;
fn print_double(x: f64) {
    print!("{}", x * two);
}
print_double(17.2);
```

它的编译产生错误：can't capture dynamic environment in a fn item。动态环境是指在定义函数的地方碰巧有效的变量集。例如，在前面的示例中，变量 two 在定义 print_double 函数时仍然定义。访问该动态环境变量的能力称为捕获环境。

相反，这是有效的：

```rust
const TWO: f64 = 2.;
fn print_double(x: f64) {
    print!("{}", x * TWO);
}
print_double(17.2);
```

and this is too:

```rust
static TWO: f64 = 2.;
fn print_double(x: f64) {
    print!("{}", x * TWO);
}
print_double(17.2);
```

这段代码是有效的，因为 const 项和静态项可以被函数捕获，但变量不能。

这样的限制有一个很好的理由：常量项和静态项具有固定值，而变量的值可以改变。所以，如果一个函数不能访问外部变量，它的行为只有在它的参数值改变时才会改变。访问外部变量的能力有效地将它们引入函数的编程接口；但是，它们在函数签名中并不明显，因此它们会误导理解代码。

但是当一个函数只能在它被定义的地方被调用时，它访问外部变量的事实并不会导致它变得更难理解，因为这些外部变量在函数被调用的地方无论如何都是可用的。

因此，我们所需功能的要求如下：内联匿名函数，具有类型推断；单个表达式作为主体；以及任何有效变量的捕获。

### Closures (闭包)

因为它的用处很大，在Rust中其实有这样一个特性，叫做闭包。
闭包只是一种更方便的函数，适合定义小的匿名函数，并在它们被定义的地方调用它们。
其实也可以定义一个闭包；将它分配给一个变量，所以给它一个名字；然后稍后使用它的名称调用它。不过，这不是闭包最典型的用法。甚至类型注释也是可能的。

这是之前的降序排序示例，将 desc 函数替换为闭包，并使代码尽可能与前一个相似

```rust
let mut arr = [4, 8, 1, 10, 0, 45, 12, 7];
use std::cmp::Ordering;
let desc = |a: &i32, b: &i32| -> Ordering {
    if a < b { Ordering::Greater }
    else if a > b { Ordering::Less }
    else { Ordering::Equal }
};
arr.sort_by(desc);
print!("{:?}", arr);
```

到目前为止，没有任何优势，但我们看到闭包可以在必须使用的地方定义，并且类型和大括号是可选的。因此，之前的代码可以转换成这个：

```rust
let mut arr = [4, 8, 1, 10, 0, 45, 12, 7];
use std::cmp::Ordering;
arr.sort_by(|a, b|
    if a < b { Ordering::Greater }
    else if a > b { Ordering::Less }
    else { Ordering::Equal });
print!("{:?}", arr);
```

Rust 闭包非常有效，因为它们不分配堆内存，而且它们通常由编译器内联扩展，从而消除所有临时对象。它们实际上与 C++ lambda 非常相似。

### Closure Invocation Syntax 闭包调用语法

这是另一个例子，展示了调用闭包的六种方法：

``` rust
let factor = 2;
let multiply = |a| a * factor;
print!("{}", multiply(13));
let multiply_ref = &multiply;
print!(
    " {} {} {} {} {}",
    (*multiply_ref)(13),
    multiply_ref(13),
    (|a| a * factor)(13),
    (|a: i32| a * factor)(13),
    |a| -> i32 { a * factor }(13));
```

这个程序包含六个相同的闭包调用。他们每个人都执行以下步骤：
* 它需要一个名为 a 的 i32 参数
* 它将参数乘以捕获的变量factor，其值为 2
* 它返回这种乘法的结果

当闭包包含多个语句时，也需要大括号，例如在这种情况下：

``` rust
print!(
    "{}",
    (|v: &Vec<i32>| {
        let mut sum = 0;
        for i in 0..v.len() {
            sum += v[i];
        }
        sum
    })(&vec![11, 22, 34]));
```
