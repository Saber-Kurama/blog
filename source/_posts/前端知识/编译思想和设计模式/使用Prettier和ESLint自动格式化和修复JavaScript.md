
[# 使用Prettier和ESLint自动格式化和修复JavaScript](https://blog.logrocket.com/using-prettier-eslint-automate-formatting-fixing-javascript/)

## Prettier VS ESLint

Prettier 可以用来格式化代码，除了格式化 `js` 和 `ts` 代码，还可以美化 `css`等
Eslint 是代码质量的检测和修复

### 什么是Prettier

Prettier是一个用于JavaScript和其他流行语言的固执己见的代码格式化程序。Prettier通过解析代码并使用自己的规则（考虑最大行长度）重新打印代码来强制执行一致的格式，并在必要时对代码进行换行。

此重写过程可防止开发人员引入任何格式错误。

创建Prettier的主要原因是为了消除对代码风格的争论。这个想法是，Prettier的风格指南是全自动的。即使Prettier没有按照您喜欢的方式100%地格式化您的代码，但为了方法的简单性，这是值得的。

虽然使用Prettier的一个重要原因是完全避免配置，但Prettier确实支持自己的配置文件，该文件具有[一些格式选项](https://prettier.io/docs/en/options.html)。

那么，为什么还有其他选项呢？

这主要是由于历史的原因。在Prettier的婴儿期添加了一些规则来吸引更多的人使用它，由于需求而添加了一些选项，由于兼容性原因而添加了一些规则。

底线是开发团队打算从现在开始再也不添加更多的选项; 你可以在Prettier的[期权理念](https://prettier.io/docs/en/option-philosophy.html)。

### 什么是ESLint


### ESLint和Prettier之间的差异


## 管理ESLint的规则以避免与Prettier发生冲突

linting 规则有两大类：格式化规则和代码质量规则。
格式化规则是影响代码风格的规则，与错误无关。例如，ESLint 中的 no-mixed-spaces-and-tabs 规则确保只有制表符或空格用于缩进。
