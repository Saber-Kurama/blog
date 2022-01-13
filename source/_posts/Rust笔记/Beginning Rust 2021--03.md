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