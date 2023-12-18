
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



为什么被叫做类型体操

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


支持哪些类型和类型运算

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

## TypeScript 类型系统中的类型运算

我们知道了 TypeScript 类型系统里有哪些类型，那么可以对这些类型做什么类型运算呢？

### 条件：extends ? :

```
type res = 1 extends 2 ? true : false;
```

```
type isTwo<T> = T extends 2 ? true: false;

type res = isTwo<1>;
type res2 = isTwo<2>;
```

高级类型的特点是传入类型参数，经过一系列类型运算逻辑后，返回新的类型。

### 推导：infer

如何提取类型的一部分呢？答案是 infer。


```ts
// 这里的泛型是类型的泛型？
type First<Tuple extends unknown[]> = Tuple extends [infer T,...infer R] ? T : never;

type res = First<[1,2,3]>;
```

### 联合：｜

联合类型（Union）类似 js 里的或运算符 |，但是作用于类型，代表类型可以是几个类型之一。

``` ts
type Union = 1 | 2 | 3;
```

### 交叉：&

交叉类型（Intersection）类似 js 中的与运算符 &，但是作用于类型，代表对类型做合并。

``` ts
type ObjType = {a: number } & {c: boolean};

```

注意，同一类型可以合并，不同的类型没法合并，会被舍弃：

### 映射类型

对象、class 在 TypeScript 对应的类型是索引类型（Index Type），那么如何对索引类型作修改呢？

```ts
type MapType<T> = {
  [Key in keyof T]?: T[Key]
}
```

``` ts
type MapType<T> = {
    [Key in keyof T]: [T[Key], T[Key], T[Key]]
}

type res = MapType<{a: 1, b: 2}>;
```


```ts
type MapType<T> = {
    [
        Key in keyof T 
            as `${Key & string}${Key & string}${Key & string}`
    ]: [T[Key], T[Key], T[Key]]
}

type atype = (string | number | symbol ) & string; //atype === string
```
``

套路一：模式匹配做提取


## 模式匹配

```ts
type p = Promise<'guang'>;

```

我们想提取 value 的类型，可以这样做：

```ts
type GetValueType<P> = P extends Promise<infer Value> ? Value : never
```


```ts
type GetValueType<P> = P extends Promise<infer Value> ? Value : never;
type GetValueResult = GetValueType<Promise<'guang'>>;
```

###  数组类型

#### First

```ts
 type arr = [1, 2, 3];
```

```ts
type GetFirst<Arr extneds unknown[]> = Arr extends [infer Frist, ...unknown[]] ? First : never;
```

> any 和 unknown 的区别： any 和 unknown 都代表任意类型，但是 unknown 只能接收任意类型的值，而 any 除了可以接收任意类型的值，也可以赋值给任意类型（除了 never）。类型体操中经常用 unknown 接受和匹配任何类型，而很少把任何类型赋值给某个类型变量。

#### Last

```ts
type GetLast<Arr extends unknownp[]> = Arr extends[...unknown[], infer Last] ? Last : never;
```

#### PopArr

我们分别取了首尾元素，当然也可以取剩余的数组，比如取去掉了最后一个元素的数组

``` ts
type PopArr<Arr extends unknown[]> = Arr extends [] ? [] : Arr extends [...infer Rest, unknown] ? Rest : never;
```

#### ShiftArr

```ts
type ShiftArr<Arr extends unknown[]> = 
    Arr extends [] ? [] 
        : Arr extends [unknown, ...infer Rest] ? Rest : never;
```


### 字符串

#### StartsWith
判断字符串是否以某个前缀开头，也是通过模式匹配：

```ts
type StartsWith<Str extends string, Prefix extends string> = 
    Str extends `${Prefix}${string}` ? true : false;
```

#### Replace

``` ts
type ReplaceStr<
    Str extends string,
    From extends string,
    To extends string
> = Str extends `${infer Prefix}${From}${infer Suffix}` 
        ? `${Prefix}${To}${Suffix}` : Str;
```

#### Trim

不过因为我们不知道有多少个空白字符，所以只能一个个匹配和去掉，需要递归。
先实现 TrimRight:

```ts
type TrimStrRight<Str extends string> = 
    Str extends `${infer Rest}${' ' | '\n' | '\t'}` 
        ? TrimStrRight<Rest> : Str;
```

同理可得 TrimLeft：

```ts
type TrimStrLeft<Str extends string> = 
    Str extends `${' ' | '\n' | '\t'}${infer Rest}` 
        ? TrimStrLeft<Rest> : Str;
```

TrimRight 和 TrimLeft 结合就是 Trim：

```ts
type TrimStr<Str extends string> =TrimStrRight<TrimStrLeft<Str>>;
```

### 函数

函数同样也可以做类型匹配，比如提取参数、返回值的类型。

#### GetParameters

```ts
type GetParameters<Func extends function> = Func extends (...args: infer Args) => unknown ? Args : never;
```

#### GetReturnType

```ts
type GetReturnType <Func extends function> = Func extends(...args: any[]) => infer ReturnType ? ReturnType : never;
```

#### GetThisParameterType

可以在方法声明时指定 this 的类型：

```ts
class Dong {
    name: string;

    constructor() {
        this.name = "dong";
    }

    hello(this: Dong) {
        return 'hello, I\'m ' + this.name;
    }
}
```

这里的 this 类型同样也可以通过模式匹配提取出来：

```ts
type GetThisParameterType<T> 
    = T extends (this: infer ThisType, ...args: any[]) => any 
        ? ThisType 
        : unknown;
```

### 构造器

构造器和函数的区别是，构造器是用于创建对象的，所以可以被 new。


####  GetInstanceType

```ts
interface Person {
    name: string;
}

interface PersonConstructor {
    new(name: string): Person;
}
```