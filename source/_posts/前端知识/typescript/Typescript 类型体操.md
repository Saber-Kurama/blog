
01. 课程介绍
2. 如何阅读本小册
3. 为什么说 TypeScript 的火爆是必然？
4. TypeScript 类型编程为什么被叫做类型体操？
5. TypeScript 类型系统支持哪些类型和类型运算？
6. 套路一：模式匹配做提取
7. 套路二：重新构造做变换
8. 套路三：递归复用做循环
9. 套路四：数组长度做计数
10. 套路五：联合分散可简化
11. 套路六：特殊特性要记清
12. 类型体操顺口溜
13. TypeScript 内置的高级类型有哪些？
14. 真实案例说明类型编程的意义
15. 类型编程综合实战一
16. 类型编程综合实战二
17. 原理篇：逆变、协变、双向协变、不变
18. 原理篇：编译 ts 代码用 tsc 还是 babel？
19. 原理篇：实现简易 TypeScript 类型检查
20. 原理篇：如何阅读 TypeScript 源码
21. 原理篇：一些特殊情况的说明22. 小册总结



简单类型系统 > 支持泛型的类型系统 > 支持类型编程的类型系统

? 针对泛型的限定？

``` ts
function getPropValue<
    T extends object,
    Key extends keyof T
>(obj: T, key: Key): T[Key] {
    return obj[key];
}
```


```ts
type ParseParam<Param extends string> =

Param extends `${infer Key}=${infer Value}`

? {

[K in Key]: Value

} : {};

  

type MergeValues<One, Other> =

One extends Other

? One

: Other extends unknown[]

? [One, ...Other]

: [One, Other];

  

type MergeParams<

OneParam extends Record<string, any>,

OtherParam extends Record<string, any>

> = {

[Key in keyof OneParam | keyof OtherParam]:

Key extends keyof OneParam

? Key extends keyof OtherParam

? MergeValues<OneParam[Key], OtherParam[Key]>

: OneParam[Key]

: Key extends keyof OtherParam

? OtherParam[Key]

: never

}

type ParseQueryString<Str extends string> =

Str extends `${infer Param}&${infer Rest}`

? MergeParams<ParseParam<Param>, ParseQueryString<Rest>>

: ParseParam<Str>;
```


TypeScript 的类型系统是`图灵完备`的，也就是能描述各种可计算逻辑。简单点来理解就是循环、条件等各种 JS 里面有的语法它都有，JS 能写的逻辑它都能写。

对类型参数的编程是 TypeScript 类型系统最强大的部分，可以实现各种复杂的类型计算逻辑，是它的优点。但同时也被认为是它的缺点，因为除了业务逻辑外还要写很多类型逻辑。


## TypeScript 类型系统中的类型

 number、boolean、string、object、bigint、symbol、undefined、null,  Number、Boolean、String、Object、Symbol

元组:  `Tuple`

接口:  `Interface`

``` ts
// 对象
interface IPerson {
    name: string;
    age: number;
}

// 函数
interface SayHello {
    (name: string): string;
}

// 构造器
interface PersonConstructor {
    new (name: string, age: number): IPerson;
}

function createPerson(ctor: PersonConstructor):IPerson {
    return new ctor('guang', 18);
}
// 对象类型、class 类型在 TypeScript 里也叫做索引类型，也就是索引了多个元素的类型的意思。对象可以动态添加属性，如果不知道会有什么属性，可以用可索引签名：
interface IPerson {
    [prop: string]: string | number;
}
const obj:IPerson = {};
obj.name = 'guang';
obj.age = 18;

```

### 枚举

```ts 
enum Transpiler {
    Babel = 'babel',
    Postcss = 'postcss',
    Terser = 'terser',
    Prettier = 'prettier',
    TypeScriptCompiler = 'tsc'
}

const transpiler = Transpiler.TypeScriptCompiler;
```

### 字面量类型

```ts
'aaa', 1, {a: 1}
//模版字面量?
aaa${string}
// 要求以#开头
function fun (str: `#${string}`) {
	console.log(str);
}
fun("a"); // error
fun("#a");
```

还有四种特殊的类型：void、never、any、unknown：

*  never 代表不可达，比如函数抛异常的时候，返回值就是 never。
- void 代表空，可以是 undefined 或 never。
- any 是任意类型，任何类型都可以赋值给它，它也可以赋值给任何类型（除了 never）。
- unknown 是未知类型，任何类型都可以赋值给它，但是它不可以赋值给别的类型。


### 类型的装饰

除了描述类型的结构外，TypeScript 的类型系统还支持描述类型的属性，比如是否可选，是否只读等：

```ts
interface IPerson {
    readonly name: string;
    age?: number;
}

type tuple = [string, number?];
```