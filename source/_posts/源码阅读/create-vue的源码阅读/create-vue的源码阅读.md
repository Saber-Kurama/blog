# create-vue的源码阅读

## 参照
[Vue 团队公开快如闪电的全新脚手架工具 create-vue，未来将替代 Vue-CLI，才300余行代码，学它！](https://juejin.cn/post/7018344866811740173)


## 使用 npm intall vue@next 初始化vue3 项目

``` shell
npm init vue@3
```

```shell
pnpm init vue@3
```

### npm init && npx
npm init 用法

``` sh
npm init [--force|-f|--yes|-y|--scope]
npm init <@scope> (same as `npx <@scope>/create`)
npm init [<@scope>/]<name> (same as `npx [<@scope>/]create-<name>`)

```
`npm init <initializer>` 时转换成`npx`命令：

-   npm init foo -> npx create-foo
-   npm init @usr/foo -> npx @usr/create-foo
-   npm init @usr -> npx @usr/create

所以

`npm init vue@3` === `npx create-vue@3`

## 关于pacakge.json

``` json
"bin": {
    "create-vue": "outfile.cjs"
},

"scripts": {
    "prepare": "husky install",
    "format": "prettier --write .",
    "build": "zx ./scripts/build.mjs",
    "snapshot": "zx ./scripts/snapshot.mjs",
    "pretest": "run-s build snapshot",
    "test": "zx ./scripts/test.mjs",
    "prepublishOnly": "zx ./scripts/prepublish.mjs"
  },
```

### run-s
`run-s` 是 [npm-run-all](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmysticatea%2Fnpm-run-all%2Fblob%2FHEAD%2Fdocs%2Frun-s.md "https://github.com/mysticatea/npm-run-all/blob/HEAD/docs/run-s.md") 提供的命令。`run-s build snapshot` 命令相当于 `npm run build && npm run snapshot`。

### zx
[zx](https://www.npmjs.com/package/zx)A tool for writing better scripts.

```js
#!/usr/bin/env zx

await $`cat package.json | grep name`

let branch = await $`git branch --show-current`
await $`dep deploy --branch=${branch}`

await Promise.all([
  $`sleep 1; echo 1`,
  $`sleep 2; echo 2`,
  $`sleep 3; echo 3`,
])

let name = 'foo bar'
await $`mkdir /tmp/${name}`
```

可以在文件中头添加
``` sh
#!/usr/bin/env zx
```
也可以
``` sh
zx ./script.mjs
```

## 使用 vscode 插件CodeTour 来做阅读代码的

## 阅读 index.js 主流程

### 解析命令参数

#### minimist

``` sh
$ node example/parse.js -a beep -b boop
{ _: [], a: 'beep', b: 'boop' }

$ node example/parse.js -x 3 -y 4 -n5 -abc --beep=boop foo bar baz
{ _: [ 'foo', 'bar', 'baz' ],
  x: 3,
  y: 4,
  n: 5,
  a: true,
  b: true,
  c: true,
  beep: 'boop' }

```

### 交互提示

#### prompts


### 输入和输出的颜色

#### kolorist

``` ts
import { red, cyan } from 'kolorist';

console.log(red(`Error: something failed in ${cyan('my-file.js')}.`));
```

### ./utils/banner.js

``` ts
const banner =
  '\x1B[38;2;66;211;146mV\x1B[39m\x1B[38;2;66;211;146mu\x1B[39m\x1B[38;2;66;211;146me\x1B[39m\x1B[38;2;66;211;146m.\x1B[39m\x1B[38;2;66;211;146mj\x1B[39m\x1B[38;2;67;209;149ms\x1B[39m \x1B[38;2;68;206;152m-\x1B[39m \x1B[38;2;69;204;155mT\x1B[39m\x1B[38;2;70;201;158mh\x1B[39m\x1B[38;2;71;199;162me\x1B[39m \x1B[38;2;72;196;165mP\x1B[39m\x1B[38;2;73;194;168mr\x1B[39m\x1B[38;2;74;192;171mo\x1B[39m\x1B[38;2;75;189;174mg\x1B[39m\x1B[38;2;76;187;177mr\x1B[39m\x1B[38;2;77;184;180me\x1B[39m\x1B[38;2;78;182;183ms\x1B[39m\x1B[38;2;79;179;186ms\x1B[39m\x1B[38;2;80;177;190mi\x1B[39m\x1B[38;2;81;175;193mv\x1B[39m\x1B[38;2;82;172;196me\x1B[39m \x1B[38;2;83;170;199mJ\x1B[39m\x1B[38;2;83;167;202ma\x1B[39m\x1B[38;2;84;165;205mv\x1B[39m\x1B[38;2;85;162;208ma\x1B[39m\x1B[38;2;86;160;211mS\x1B[39m\x1B[38;2;87;158;215mc\x1B[39m\x1B[38;2;88;155;218mr\x1B[39m\x1B[38;2;89;153;221mi\x1B[39m\x1B[38;2;90;150;224mp\x1B[39m\x1B[38;2;91;148;227mt\x1B[39m \x1B[38;2;92;145;230mF\x1B[39m\x1B[38;2;93;143;233mr\x1B[39m\x1B[38;2;94;141;236ma\x1B[39m\x1B[38;2;95;138;239mm\x1B[39m\x1B[38;2;96;136;243me\x1B[39m\x1B[38;2;97;133;246mw\x1B[39m\x1B[38;2;98;131;249mo\x1B[39m\x1B[38;2;99;128;252mr\x1B[39m\x1B[38;2;100;126;255mk\x1B[39m'

export default banner
```

这一段是  ANSI escape code

[ANSI escape code 语法颜色笔记](http://noyobo.com/2015/11/13/ANSI-escape-code.html)

### 如果设置了 feature flags 跳过 prompts 询问

是否使用 --vitest 这种标签

``` ts
 const isFeatureFlagsUsed =
    typeof (
      argv.default ||
      argv.ts ||
      argv.jsx ||
      argv.router ||
      argv.pinia ||
      argv.tests ||
      argv.vitest ||
      argv.cypress ||
      argv.eslint
    ) === 'boolean'
```

### prompts的询问配置

```
 // Prompts:
    // - Project name: 
    //   - whether to overwrite the existing directory or not?
    //   - enter a valid package name for package.json
    // - Project language: JavaScript / TypeScript
    // - Add JSX Support?
    // - Install Vue Router for SPA development?
    // - Install Pinia for state management?
    // - Add Cypress for testing?
    // - Add ESLint for code quality?
    // - Add Prettier for code formatting?
```

``` ts
function isValidPackageName(projectName) {
  return /^(?:@[a-z0-9-*~][a-z0-9-*._~]*\/)?[a-z0-9-~][a-z0-9-._~]*$/.test(projectName)
}

function toValidPackageName(projectName) {
  return projectName
    .trim()
    .toLowerCase()
    .replace(/\s+/g, '-')
    .replace(/^[._]/, '')
    .replace(/[^a-z0-9-~]+/g, '-')
}

function canSafelyOverwrite(dir) {
  return !fs.existsSync(dir) || fs.readdirSync(dir).length === 0
}
```

### 根据模板文件生成初始化项目所需文件

``` ts
renderTemplate 文件中
  // package.json 的处理 依赖
  if (filename === 'package.json' && fs.existsSync(dest)) {
    // merge instead of overwriting
    const existing = JSON.parse(fs.readFileSync(dest))
    const newPackage = JSON.parse(fs.readFileSync(src))
    const pkg = sortDependencies(deepMerge(existing, newPackage))
    fs.writeFileSync(dest, JSON.stringify(pkg, null, 2) + '\n')
    return
  }

// `.`文件的处理
  if (filename.startsWith('_')) {
    // rename `_file` to `.file`
    dest = path.resolve(path.dirname(dest), filename.replace(/^_/, '.'))
  }
```