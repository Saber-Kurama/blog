# 设置 GraphQL 使用 Express.js

我们前端的基本设置和原型现在已经完成。现在，我们需要得到我们的 GraphQL 服务器正在运行以开始实现后端。 Apollo 和 Express.js将用于构建我们后端的基础。

本章将解释 Express.js 的安装过程，以及配置对于我们的 GraphQL 端点。我们将快速浏览所有基本功能Express.js 和我们后端的调试工具。

本章涵盖以下主题：
* Express.js 安装及说明
* Express.js 中的路由
* Express.js 中的中间件
* 将 Apollo 服务器绑定到 GraphQL 端点
* 发送我们的第一个 GraphQL 请求
* 后端调试和日志记录


https://github.com/PacktPublishing/Full-Stack-Web-Development-with-GraphQL-and-React-Second-Edition/tree/main/Chapter02

## Getting started with Node.js and Express. js（Node.js 和 Express.js 入门）

本书的主要目标之一是建立一个 GraphQL API，然后由我们的 React 前端消耗。接受网络请求——尤其是 GraphQL请求——我们将设置一个 Node.js 网络服务器。

Node.js Web 服务器领域最重要的竞争对手是 Express.js、Koa 和Hapi。在本书中，我们将使用 Express.js。大多数关于 Apollo 的教程和文章依靠它

Express.js 也是最常用的 Node.js Web 服务器，并将自己描述为Node.js Web 框架，提供构建 Web 应用程序所需的所有主要功能

安装 Express.js 很容易。我们可以像在上一章使用npm：

```
npm install --save express
```