---
 title: Rust 2021 版入门
 date: 2022-01-13
 tag: rust
---
# rust
## Enumerations and Matching  枚举和匹配
* How enums help in defining variables that can take on values only from a finite set of cases (枚举如何帮助定义只能从有限的案例集中取值的变量)
* How enums can be used to implement discriminated union types (如何使用枚举来实现有区别的联合类型)
* How to use the match pattern-matching construct to handle enums (如何使用匹配模式匹配结构来处理枚举)
* How to use the match construct to handle other data types, like integer numbers, strings, and single characters (如何使用匹配结构来处理其他数据类型，如整数、字符串和单个字符)
* How to define enumerations containing fields of data and how to match them with literals (如何定义包含数据字段的枚举以及如何将它们与文字匹配)
* How to define variables in patterns of match statements (如何在匹配语句模式中定义变量)
* How to use match expressions (如何使用匹配表达式)
* How to use Boolean guards to generalize the pattern-matching of the match construct (如何使用布尔守卫来概括匹配结构的模式匹配)
* How to use the if-let and the while-let constructs (如何使用 if-let 和 while-let 结构)

### Enumerations 枚举
``` rust
#[allow(dead_code)]
enum Continent {
    Europe,
    Asia,
    Africa,
    America,
    Oceania,
}
let contin = Continent::Asia;
match contin {
    Continent::Europe => print!("E"),
    Continent::Asia => print!("As"),
    Continent::Africa => print!("Af"),
    Continent::America => print!("Am"),
    Continent::Oceania => print!("O"),
}
```
enum”关键字引入了新的 Continent 类型，紧随其后指定。这种类型称为枚举，因为它列出了一组项目，在内部将唯一的编号与每个项目相关联。在示例中，Continent 类型的允许值为 Europe、Asia、Africa、America 和 Oceania，它们在内部分别由值 0u8、1u8、2u8、3u8 和 4u8 表示。这样的允许值被命名为变体。
编译器包含一个有用的检查：每个变体必须在应用程序中至少构造一次，否则它是无用的，编译器会发出警告。在前面的程序中，只构建了 Asia 变体，因此编译器会发出四个关于未使用变体的警告。为了避免它们，添加了属性#[allow(dead_code)]。实际上，枚举变体的创建是某种代码。编译器认为这段代码永远不会运行，所以它是死代码。我们被警告存在这种死代码。
请注意，变体的使用必须由其类型的名称来限定，就像在 Continent::Asia 中一样。

### The match Construct （ match 构造 ）

“=>”（称为胖箭头）
事实上，任何有效的表达式都会变成有效的语句，如果你在它后面加上一个分号字符

``` rust
let a = 7.2;
12;
true;
4 > 7;
5.7 + 5. * a;
```
这段代码是有效的，当然，它什么也没做。实际上，对于最后两个语句，会生成以下警告：必须使用未使用的比较，以及必须使用未使用的算术运算。发生这种情况是因为编译器认为这种毫无意义的表达式可能是编程错误。为避免这些警告，请编写此代码

``` rust
let a = 7.2;
12;
true;
let _ = 4 > 7;
let _ = 5.7 + 5. * a;
```

可以使用语句做更多分支做更多事情
```rust
#[allow(dead_code)]
enum Continent {
    Europe,
    Asia,
    Africa,
    America,
    Oceania,
}
let mut contin = Continent::Asia;
match contin {
    Continent::Europe => {
        contin = Continent::Asia;
        print!("E");
    },
    Continent::Asia => { let a = 7; print!("{}", a); }
    Continent::Africa => print!("Af"),
    Continent::America => print!("Am"),
    Continent::Oceania => print!("O"),
}
```

### Relational Operators and Enums  关系运算符和枚举

“使用 `==` 运算符无法比较枚举
对于枚举，不仅禁止使用`==`运算符，还禁止使用其他关系运算符：“!=”、“<”、“<=”、“>”和“>=”
下面语句出错

``` rust
enum CardinalPoint { North, South, West, East }
let direction = CardinalPoint::South;
if direction == CardinalPoint::North { }
if CardinalPoint::South < CardinalPoint::North { }
```
### Handling All the Cases 处理所有的case 穷尽

可以使用 `_`, 下划线符号总是匹配任何值，所以它避免了编译错误，因为这样所有的情况都被处理了。当然，这样一个“包罗万象”的case必须是最后一个
``` rust
match direction {
    CardinalPoint::North => print!("NORTH"),
    CardinalPoint::South => print!("SOUTH"),
    _ => {},
}
```
`_`模式对应于 C 语言的“默认”情况

###  Using match with Numbers 使用数字匹配
match 结构除了需要枚举之外，还可以用于其他数据类型
``` rust
match "value" {
    "val" => print!("value "),
    _ => print!("other "),
}
match 3 {
    3 => print!("three "),
    4 => print!("four "),
    5 => print!("five "),
    _ => print!("other "),
}
match '.' {
    ':' => print!("colon "),
    '.' => print!("point "),
    _ => print!("other "),
}
```

###  Enumerations with Data 带数据的枚举

Rust 枚举并不总是像以前看到的那样简单。这是一个更复杂的例子
```rust
#[allow(dead_code)]
enum Result {
    Success(u8),
    Failure(u16, char),
    Uncertainty,
}
// let outcome = Result::Success(1);
let outcome = Result::Failure(20, 'X');
match outcome {
    Result::Success(0) => print!("Result: 0"),
    Result::Success(1) => print!("Result: 1"),
    Result::Success(_) => print!("Result: other"),
}
#[allow(dead_code)]
enum Result {
    Success(u8),
    Failure(u16, char),
    Uncertainty,
}
// let outcome = Result::Success(1);
let outcome = Result::Failure(20, 'X');
match outcome {
    Result::Success(0) => print!("Result: 0"),
    Result::Success(1) => print!("Result: 1"),
    Result::Success(_) => print!("Result: other"),
    Result::Failure(10, 'X') => print!("Error: 10 X"),
    Result::Failure(10, _) => print!("Error: 10"),
    Result::Failure(_, 'X') => print!("Error: X"),
    Result::Failure(_, _) => print!("Error: other"),
    Result::Uncertainty => {},
}
```

### match Statements with Variables in Patterns  将语句与模式中的变量匹配
我们看到了如何在匹配语句模式中使用文字。 Rust 还允许在这些模式中使用变量，就像在这个例子中一样
``` rust
#[allow(dead_code)]
enum Result {
    Success(u8),
    Failure(u16, char),
    Uncertainty,
}
// let outcome = Result::Success(13);
let outcome = Result::Failure(20, 'X');
match outcome {
    Result::Success(0) => print!("Result: 0"),
    Result::Success(1) => print!("Result: 1"),
    Result::Success(n) => print!("Result: {}", n),
    Result::Failure(10, 'X') => print!("Error: 10 X"),
    Result::Failure(10, m) => print!("Error: 10 in module {}", m),
    Result::Failure(code, 'X') => print!("Error: n.{} X", code),
    Result::Failure(code, module) =>
        print!("Error: n.{} in module {}", code, module),
    Result::Uncertainty => {},
}
```

### match Expressions 匹配表达式
``` rust
#[allow(dead_code)]
enum CardinalPoint { North, South, West, East }
let direction = CardinalPoint::South;
print!("{}", match direction {
    CardinalPoint::North => 'N',
    CardinalPoint::South => 'S',
    _ => '*',
});
// 打印 S。因为match也是一个表达式
```

### Use of Guards in match Constructs 使用守卫在match构造中
假设我们要将整数分为以下几类：所有负数、零数、一数和所有其他正数。这是执行此类分类的代码
``` rust
for n in -2..5 {
    println!("{} is {}.", n, match n {
        0 => "zero",
        1 => "one",
        _ if n < 0 => "negative", // 只有当布尔条件为真时，这样的子句才会导致该模式匹配。它被命名为守卫
        _ => "plural",
    });
}
```
### if-let and while-let Constructs  if-let 和 while-let 构造

有时你只需要检查一个枚举值是否是某个变体，在这种情况下你需要提取它的字段。可以通过以下代码实现
``` rust
enum E {
    Case1(u32),
    Case2(char),
    Case3(i64, bool),
}
let v = E::Case3(1234, true);
match v {
    E::Case3(n, b) => if b { print!("{}", n) }
    _ => {}
}
```
Rust 语言还支持以下语法，它替换了前面的 match 语句：
```rust
if let E::Case3(n, b) = v {
    if b { print!("{}", n) }
}
```

while let
``` rust
enum E {
    Case1(u32),
    Case2(char),
}
let mut v = E::Case1(0);
while let E::Case1(n) = v {
    print!("{}", n);
    if n == 6 { break; }
    v = E::Case1(n + 1);
}
```

## Using Heterogeneous Data Structures
