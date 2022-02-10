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
A tool for writing better scripts.

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

可以在文件中