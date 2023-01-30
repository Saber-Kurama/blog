
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
Prettier有一个 [`tabs` option](https://prettier.io/docs/en/options.html#tabs)来做同样的事情。
其次，代码质量规则提高了代码质量，可以防止或捕获错误。例如，ESLint 中的规则 no-implicit-globals 不允许全局范围变量。
从其他脚本创建的全局变量可能会发生名称冲突，这通常会导致运行时错误或意外行为。
问题是Prettier和ESLint的规则重叠，我们希望它们不重叠！
通常，我们希望 Prettier 处理第一个类别，ESLint 处理第二个类别。有些规则可能很难归为一类或另一类；我们不需要对它们属于哪个类别迂腐。
我们的兴趣在于确保 Prettier 或 ESLint 执行特定操作并且不会相互碰撞。
至于它们的运行顺序，通常最好在 ESLint 之前运行 Prettier，因为 Prettier 会从头开始重新打印您的整个程序。所以，如果你想让 ESLint 参与格式化操作，你应该在 Prettier 之后运行它以防止更改被覆盖。
如果您不熟悉ESLint和Prettier，我们将在下一节介绍如何配置和使用它们。

## ESLint和Prettier初始配置和基本用法


## Methods for linting and pretty-printing your code

### 删除冲突规则并连续运行

此方法是最干净、最有效的方法，也是推荐使用的最佳方法。

通过使用以下配置，可以很容易地关闭与ESLint中的Prettier冲突的规则：

* [eslint-config-prettier](https://github.com/prettier/eslint-config prettier) for JavaScript
* [tslint-config-prettier](https://github.com/alexjoverm/tslint-config-prettier) for TypeScript

首先，安装JavaScript的配置：

```
npm install --save-dev eslint-config-prettier
```

这里有一个例子。 `.eslintrc.json`

```
{
  // ...
  extends: [
    // ...
    'eslint:recommended',
    "prettier" // Make sure this is the last
  ],
  // ...
}
```

`package.json`

```
{
   "name": "no-worries-setup",   
   "version": "1.0.0",
   "scripts": {
    "lint": "npx eslint src test",
    "lint:fix": "npm run lint -- --fix",
    "prettier": "npx prettier src test --check",
    "prettier:fix": "npm run prettier -- --write",
    "format": "npm run prettier:fix && npm run lint:fix",
  }
  // ...
}
```

现在，您可以运行`npm run format`命令来一次性格式化和修复所有代码。

要与VS代码一起使用，请安装扩展：[ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)、[Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)和[Format Code Action](https://marketplace.visualstudio.com/items?itemName=rohit-gohri.format-code-action&ssr=false#review-details)，并更新您的用户设置（`setting.json` ），如下所示：

```
{
  //...
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "eslint.probe": [
      "javascript",
      "javascriptreact",
      "vue"
  ],
  "editor.formatOnSave": false,
  // Runs Prettier, then ESLint
  "editor.codeActionsOnSave": [
    "source.formatDocument",
    "source.fixAll.eslint"
  ],
  "vetur.validation.template": false
  // ...
}
```

首先，您需要在保存时禁用编辑器格式设置（`editor.formatOnSave` ）;我们希望通过代码操作来处理所有事情

2020年3月 [（v1.44）](https://code.visualstudio.com/updates/v1_44#_explicit-ordering-for-code-actions-on-save&quot)，`editor.codeActionsOnSave` 属性已更新为接受代码操作数组，这允许有序的代码操作。如果安装了[Format Code Action](https://marketplace.visualstudio.com/items?itemName=rohit-gohri.format-code-action&ssr=false#review-details)扩展，就可以将格式设置作为代码操作使用。

现在，我们可以按照任意顺序将Prettier和ESLint作为代码操作运行。太好了!

在本例中，我们首先运行Prettier，操作如下  `source.formatDocument`（它使用默认格式化程序），然后运行  `eslint --fix` 与 `源。全部修复。埃斯林特` 行动。