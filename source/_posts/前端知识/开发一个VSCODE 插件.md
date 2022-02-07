# 开发一个VSCODE 插件

## 参考

[vscode插件开发教程](https://www.jianshu.com/p/e642856f6044)
[从0到1开发一款自己的vscode插件](https://segmentfault.com/a/1190000040720760)
[VsCode 插件语言插件开发](https://sunra.top/2021/06/04/lsp/)
[Vscode 插件开发](https://www.ronpad.com/docs/vscode/)
[json-go-ts VS Code插件开发](https://segmentfault.com/a/1190000040386930)
[VSCode插件开发全攻略](http://blog.haoji.me/vscode-plugin-overview.html)
[VS Code 插件开发文档](https://www.bookstack.cn/read/VS-Code-Extension-Doc-ZH/README.md)

## VSCODE 插件能做什么

* 不受限的本地磁盘访问
* 自定义命令、快捷键、菜单
* 自定义跳转、自动补全、悬浮提示
* 自定义设置、自定义欢迎页
* 自定义WebView
* 自定义左侧功能面板
* 自定义颜色、图标主题
* 新增语言支持
* Markdown增强
* 其他

## 脚手架 创建项目

```
npm install -g yo generator-code
```

```
yo code
```

## 项目结构

```
├── .vscode  
│   ├── launch.json     // 插件加载和调试的配置  
│   └── tasks.json      // 配置TypeScript编译任务  
├── .gitignore          // 忽略构建输出和node_modules文件  
├── README.md           // 一个友好的插件文档  
├── src  
│   └── extension.ts    // 插件源代码  
├── package.json        // 插件配置清单  
├── tsconfig.json       // TypeScript配置

```
## 插件清单

## 代码片段  snippets

### 创建自己的片段

请在**文件**>**首选项**（macOS 上的**代码**>**首选项**）下选择**用户代码片段，然后选择代码片段应该出现的语言（按**[语言标识符](https://code.visualstudio.com/docs/languages/identifiers)），或者如果它们应该出现**新的全局片段文件**选项出现在所有语言中

``` json
// fe.code-snippets

{
	// Example:
	"Print to console": {
		"scope": "javascript,typescript",
		"prefix": "log",
		"body": [
			"console.log('${1|one,two,three|}');",
			"$2"
		],
		"description": "Log output to console"
	}
}

```