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

## Doing Arithmetic and Writing More Code(做算术)