
##  MVP原型系统： 将组件封装为组件库

技术选型

`vite` + `tsx` + `pnpm`

## CSS样式：用UnoCSS实现原子化CSS

其实不建议组件库使用 原子化css 。这会要求项目中也必须采用同类型的原子化css

## 文档建设：创建具备Demo示例功能的文档网站

`md` 文档

可以参考 `TDesign`


## 单元测试(一)： 使用Jest进行前端单元测试

### 测试前端页面

- 解决的办法有两个 ：
    - 将测试用例放到浏览器中运行；
    -  用 dom 仿真模拟一个 dom 对象。


## 单元测试 (二)： 搭建Vitest的单元测试环境

## 规范化： Eslint + Prettier + Husky

项目规范可以分为：

- 编码规范；
- 项目结构规范；
- 文件命名规范；
- git commit 版本规范；
- 工作流 workflow规范。

### 编码规范

- JS代码规范
    
    - [airbnb-中文版](https://link.juejin.cn/?target=https://github.com/lin-123/javascript "https://link.juejin.cn/?target=https://github.com/lin-123/javascript")
    - [standard (24.5k star) 中文版](https://link.juejin.cn/?target=https://github.com/standard/standard/blob/master/docs/README-zhcn.md "https://link.juejin.cn/?target=https://github.com/standard/standard/blob/master/docs/README-zhcn.md")
    - [百度前端编码规范 3.9k](https://link.juejin.cn/?target=https://github.com/ecomfe/spec "https://link.juejin.cn/?target=https://github.com/ecomfe/spec")
- CSS代码规范
    
    - [styleguide 2.3k](https://link.juejin.cn/?target=https://github.com/fex-team/styleguide/blob/master/css.md "https://link.juejin.cn/?target=https://github.com/fex-team/styleguide/blob/master/css.md")
    - [spec 3.9k](https://link.juejin.cn/?target=https://github.com/ecomfe/spec/blob/master/css-style-guide.md "https://link.juejin.cn/?target=https://github.com/ecomfe/spec/blob/master/css-style-guide.md")

### 目录规范

### 文件命名规范

### Eslint + Prettier 代码检查工具

### Git commit 提交规范


##  软件包封装： 如何发布兼容多种 JS 模块标准的软件包？

**第一，需要考虑的是需要支持哪些模块规范。**

目前常见的模块规范有： - IFFE：使用立即执行函数实现模块化 例：(function()) {}； - CJS：基于 CommonJS 标准的模块化； - AMD：使用 Require 编写； - CMD：使用 SeaJS 编写； - ESM：ES 标准的模块化方案 ( ES6 标准提出 )； - UMD：兼容 CJS 与 AMD、IFFE 规范。

其中最常用的有三类：ESM、CJS 和 IFFE。 ESM 标准目前已经是前端开发的标配，无论是选用 Webpack 还是 Vite ，都会采用这种模块规范。其次是 CJS，不可否认，有大量的存量代码还使用 CJS 规范，完全没有必要因为引入一个库去更改编译规则。最后是 IFFE 这种类型，非常适用于逻辑简单，无需搭建工程化环境的前端应用。

**第二，需要考虑的是代码的压缩和混淆问题。**

代码压缩是指去除代码中的空格、制表符、换行符等内容，将代码压缩至几行内容甚至一行，这样可以提高网站的加载速度。混淆是将代码转换成一种功能上等价，但是难以阅读和理解的形式。混淆的主要目的是增加反向工程的难度，同时也可以相对减少代码的体积，比如将变量名缩短就会减少代码的体积。

**第三，还需要考虑 SourceMap 配置。**

SourceMap 就是一个信息文件，里面存储了代码打包转换后的位置信息，实质是一个 json 描述文件，维护了打包前后的代码映射关系。通

常输出的模块不会提供 SourceMap，因为通过 sourcemap 就很容易还原原始代码。但是如果你想在浏览器中断点调试你的代码，或者希望在异常监控工具中定位出错位置，SourceMap 就非常有必要。所以还是要正确掌握 SourceMap 的生成方法。

## 持续集成 CI： 基于 github Action 的回归验证

给组件库添加自动回归验证机制。自动回归验证是前端持续集成的一个重要任务。Github Action 也是最方便的一种持续集成实现手段。除了上面这些，包括自动通知、部署测试环境等操作也都可以通过持续集成实现。如果希望搭建私有持续集成服务可以考虑使用 gitlab CI ，使用方法和 Action 类似。

最后留一些思考题帮助大家复习，也欢迎大家在评论区讨论。

- 持续集成的定义是什么？
- 工作流 workflow、任务 Job、Action 都是什么？
- 如何获得 CI 徽章？

`回归验证` 是指发布之后验证

## 开发许可证：维护自己的版权、拒绝拿来党

LICENSE 协议

## 组件发布： 建立语义化版本与提交软件包仓库 Npm

### 使用语义化版本

### 利用Action 实现自动发布


## 建立组件库生态： 利用 Monorepo 方式管理组件库生态

## 按需引入: 实现组件库的按需引入功能

`unplugin-vue-components`

## 文档部署： 用 Vercel 部署你的线上文档

## README： 编写标准的 README

- Banner + Title 居中；
- 徽章的颜色重新定制和 Logo 呼应，并且细心地使用了渐变效果。
- - 特性描述精炼准确；
- 字体图标开头提升了页面视觉效果；
- 引用链接完整清晰。


这个项目系统地讲述了 README 的编写方法。文中提到标准的 README，最基本的部分包括以下几大内容：

- Background 背景；
- Install 安装 ；
- Usage 用途；
- Badge徽章 - 项目的标准，例： npm 下载量、测试覆盖率、通过 CI 工具持续验证 ；
- Contributing 贡献者名单；
- License 代码许可证。

这个应该是一个最低配的 README。

通过这个结构可以让使用者最短时间了解并上手。

如果扩展一下，让项目介绍更加的丰满，还可以采用以下结构：

- Title；
- Banner；
- Badges；
- Short Description；
- Long Description ；
- Table of Contents；
- Security；
- Background；
- Install；
- Usage；
- Extra Sections；
- API；
- Maintainers；
- Thanks；
- Contributing；
- License 。

以上，都是给你提供的一个思维框架，在实际运用中可以根据实际情况灵活掌握。

## 品质保证：发布覆盖率测试报告

代码覆盖率（Code coverage）是软件测试中的一种度量指标，描述测试过程中（运行时）被执行的源代码占全部源代码的比例。

