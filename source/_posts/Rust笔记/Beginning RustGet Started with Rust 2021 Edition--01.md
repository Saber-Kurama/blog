---
 title: Rust 2021 版入门
 date: 2022-01-09
 tag: rust
---

#  rust

## Getting Started （入门）

*  How to write and run your first program in the Rust language
*  How to print text and numbers on 
*  How to write comments in your code

``` rust
fn main () {}
```

### Hello, World!
``` rust
fn main() {
	print!(" Hello, world!");
}
```
`print` 是一个宏定义的名称(macro defined), `!`字符表示前面的名称是一个宏。宏类似函数，但是宏是一些rust的代码，宏调用和函数的调用区别在于，函数调用执行函数定义中的代码，宏调用则要求编译器用宏定义的主体替换宏调用

### Printing Combinations of Literal String (打印字面量字符串的组合)

``` rust
print!("{}, {}!", "Hello", "world");
```
下面两种都会错误
``` rust
print!("{}!", "Hello", "world");
print!("{}, {}!", "hello");
```
### Print Several Lines of Text ( 打印多行文本)

``` rust
print!("“First line\nSecond line\nThird line\n");
```
### Printing Interger Numbers
``` rust
print!("my number : {}", 140);
print!("my number1: {}", 000140);
```
### Comments (注释)
``` rust
// This program

// prints a number.

print!("{}", 34); // thirty-four

/* print!("{}", 80);

*/```
rust 程序员避免在生产代码中使用多行代码注释，只使用单行注释，多行注释只是为了经一些代码暂时排除在编译之外
单行注释的优点： 使用搜索工期提取源代码的行的时候，或者在比对的时候不同版本的时候，立即清楚那些是注释，那些不是

rust 的注释是可以嵌套的

## Doing Arithmetic and Writing More Code(做算术写更多的代码)
*  How to compute an arithmetic operation between integer numbers or between floating-point numbers
* How to write a program containing several statements
* How to print strings in several lines

### 整数相加
``` rust
print!("{} + {} = {}", 34, 80, 80 + 34);
```
构建的时候，会将`80 + 34` 这个表达式 ，二进制数和加法指令存储到可执行文件中，但是由于数字是常量，构建优化就会直接变成`114`

### Other Operations Between Integer Numbers（整数的其他运算）

`+ ` , `-`,  `*`, `/`, `%` 

### Floating-Point Arithmetic

``` rust
print!("THe sum is {}", 80.3 + 34.8); // 115.1
print!("The sum is {}", 80.3 + 34.9); // 115.1999999999999
```
rust 中浮点数和整数的混合运算必须显示转换
``` rust
pirnt!("then sum is {}", 2.7 + 1.);
```
### Sequences of Statements (语句序列)

rust 的的一些风格规范
* 在函数内将行缩进4个空格（？）
* 避免在语句中添加几个连续空格
* 避免一行超过80，长语句分成多行

### Breaking Literal String （文字字符折行）

``` rust
println!(
        "{}",
        "This \
        is \
        just one line"
    ); // This is just one line
```

`\`  字符串折行
`\n` 字符串换行

## Naming Objects（命名对象）
*  值、对象和变量的概念
*  变量的可变性概念
*  初始化和重新赋值的区别
*  如何避免未使用变量的警告
*  布尔表达式的概念
*  编译器对赋值执行那种类型的检查
*  一些运算符如何同时执行算术运算和赋值
*  如何调用Rust标准库中定义的函数

### Associating Names to Values （将名称和值进行关联）

什么是值，什么是对象。 值是一个抽象的数学概念，可以存储在计算机内存中
包含值的内存部分被命名为“对象”
标识符-对象成为变量
声明并初始化了一个对象
这些概念和c语言中的概念一致

### Mutable Variables （可变变量）
``` rust
let mut a = 1;
a = 2;
```
rust 默认变量不可变

### Not Mutated Mutable Variables

如果对一个变量定义 mutable，但一直都没有修改赋值，就会有一个警告`warning: variable does not need to be mutable`

### Uninitialized Variables(未初始的变量)

### The Leading Underscore（下划线）

``` rust
let number = 12 ; // 如果后面没有使用该变量就会有一个警告 
```
``` rust
let _number = 12; // 就不会有警告 
```

使用下划线`_`就不提示 `warning: unused variable: `number`

``` rust
let _ = 12; //孤立的下划线就会报错或者警告
```

### Boolean Values 布尔值

 `true` 和 `false`
 关系运算法
 `==`, `!=`, `<`, `<=`, `>`, `>=`

 字符串的比较 `>`

 ### Boolean Expressions 布尔表达式

`&&` , `||`

### Type Consistency in Assignments 分配中的类型一致性

编译中都有一个类型

### Type Inference 类型推断

``` rust
let number
```
仅仅这一句话是错误的，没有类型

```rust
let _number:i32;
```
赋值之后，就可以类型推断

### Change of Type and of Mutability 类型和可变性的变化

变量遮蔽

### Compound Assignment Operators 复合赋值运算符

``` rust
let mut a = 12;
a += 1;
a -= 4;
a *= 7;
a /= 6;
print!("{}", a);
```
### 使用标准库的函数
``` rust
str::len("abcde");
```

##  Controlling Execution Flow  控制执行流程
