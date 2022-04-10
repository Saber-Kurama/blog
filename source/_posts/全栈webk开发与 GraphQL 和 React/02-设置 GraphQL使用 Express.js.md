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

### 设置 Express.js
和往常一样，我们需要一个加载了所有主要组件的根文件来组合它们到实际应用中。

### 在开发中运行 Express.js

```
"server": "nodemon --exec babel-node --watch src/server src/
server/index.js"
```

##  Express.js 中的路由

了解路由对于扩展我们的后端代码至关重要。在本节中，我们是将通过一些简单的路由示例进行播放。

通常，路由处理应用程序响应特定端点的方式和位置和方法。

在 Express.js 中，一个路径可以响应不同的 HTTP 方法，并且可以有多个处理函数。这些处理函数按照它们的顺序一一执行代码中指定。路径可以是简单的字符串，也可以是复杂的正则表达式或图案。

当您使用多个处理函数时 - 作为数组或作为多个参数——一定要通过 next到每个回调函数。 你调用next， 你将执行从一个回调函数移交给该行中的下一个函数。 这些函数也可以是中间件。 我们将在下一节中介绍这一点。

这是一个简单的例子。

```
app.get('/',
    function (req, res, next) {
        console.log('first function');
        next();
    },
    function (req, res) {
        console.log('second function');
        res.send('Hello World!');
    });
```


##  Serving our production build 服务于我们的生产构建

我们可以通过 Express.js 为前端的生产构建提供服务。 这种方法是不适合开发目的，但对于测试构建过程和查看很有用我们的实时应用程序将如何运行。

同样，将前面的路由示例替换为以下代码：
```
import path from 'path';

const root = path.join(__dirname, '../../');

app.use('/', express.static(path.join(root, 'dist/client')));
app.use('/uploads', express.static(path.join(root, 'uploads')));
app.get('/', (req, res) => {
  res.sendFile(path.join(root, '/dist/client/index.html'));
});
```

## 使用 Express.js 中间件

### 安装重要的中间件

```
npm install --save compression cors helmet
```

### Express Helmet

Helmet 是一个工具，它允许您设置各种 HTTP 标头以保护您的应用程序。


### Compression with Express.js
为 Express.js 启用压缩可以节省您和您的用户带宽，这很容易去做

>
每当您有这样的中间件或匹配相同路径的多个路由时，您都需要检查初始化顺序。 除非您运行下一个命令，否则只会执行第一个匹配的路由。 之后定义的所有路由都不会被执行。

### CORS in Express.js

## 将 Express.js 与 Apollo 相结合

First things first; we need to install the Apollo and GraphQL dependencies:

```
npm install --save apollo-server-express graphql @graphql-tools/schema
```

Apollo 提供了一个特定于 Express.js 的包，该包将自身集成到 Web 服务器中。还有一个没有 Express.js 的独立版本。Apollo 允许您使用可用的 Express.js 中间件。在某些情况下，您可能需要提供非 GraphQL 路由到未实现 GraphQL 或无法理解 JSON 响应的专有客户端。仍然有理由为 GraphQL 提供一些后备方案。 在这些情况下，您可以依赖 Express.js，因为您已经在使用它。

为服务创建一个单独的文件夹。 服务可以是 GraphQL 或其他路由：

```
mkdir src/server/services/
mkdir src/server/services/graphql
```

在 graphql 文件夹中创建一个 index.js 文件，作为我们 GraphQL 服务的起点。它必须为初始化处理多个事情.让我们一一浏览它们并将它们添加到 index.js 文件中：
1.  First, we must import theapollo-server-expressand@graphql-tools/schemapackages:
```
 import { ApolloServer } from 'apollo-server-express';
 import { makeExecutableSchema } from '@graphql-tools/schema' 
```

2.   接下来，我们必须将 GraphQL 模式与解析器函数结合起来。我们必须在顶部从单独的文件中导入相应的模式和解析器函数。 GraphQL 模式是 API 的表示，即客户端可以请求或运行的数据和函数。解析器函数是模式的实现。两者都需要匹配。 您不能返回字段或运行不在架构内的突变：
```
import Resolvers from './resolvers';
import Schema from './schema';
```


3.   @graphql-tools/schema 的 makeExecutableSchema 函数包合并了 GraphQL 模式和解析器函数，解析我们将要写入的数据。当您定义不在模式中的查询或突变时，makeExecutableSchema 函数会引发错误。生成的模式由我们的 GraphQL 服务器执行，解析数据或运行我们请求的突变：
```
const executableSchema = makeExecutableSchema({typeDefs: Schema,resolvers: Resolvers});
```

4. 我们将其作为schema参数传递给 Apollo 服务器。context 属性包含 Express.js 的request对象。在我们的解析器函数中，如果需要，我们可以访问请求：
```
const server = new ApolloServer({schema: executableSchema,context: ({ req }) => req});
```

5.  此 index.js 文件导出初始化的服务器对象，该对象处理所有 GraphQL 请求：

```
export default server;
```

现在我们正在导出 Apollo 服务器，需要将其导入其他地方。我发现在服务层上有一个 index.js 文件很方便，这样我们只有在添加新服务时才依赖这个文件。

在 services 文件夹中创建 index.js 文件并输入以下代码：

```
import graphql from './graphql';

export default {
  graphql,
};

```

前面的代码需要我们来自 graphql 文件夹的 index.js 文件，并将所有服务重新导出到一个大对象中。 如果需要，我们可以在这里定义更多服务。

为了让我们的客户可以公开访问我们的 GraphQL 服务器，我们将把 Apollo 服务器绑定到 /graphql 路径

将服务 index.js 文件导入到 server/index.js 文件中，如下：

```
import services from './services';
```

services 对象仅保存 graphql 索引。 现在，我们必须使用以下代码将 GraphQL 服务器绑定到 Express.js Web 服务器：

```
const serviceNames = Object.keys(services);

for (let i = 0; i < serviceNames.length; i += 1) {
  const name = serviceNames[i];
  if (name === 'graphql') {
    (async () => {
      await services[name].start();
      services[name].applyMiddleware({ app });
    })();
  } else {
    app.use(`/${name}`, services[name]);
  }
}
```


### Writing your first GraphQL schemas
让我们首先在 graphql 文件夹中创建一个 schema.js。 您还可以将多个较小的模式拼接成一个更大的模式。 当您的应用程序、类型和字段增长时，这将更清晰并且有意义。 对于这本书，一个文件就可以了，我们可以将以下代码插入到 schema.js 文件中：

前面的代码代表了一个基本模式，它至少能够为第 1 章准备开发环境中的假帖子数组提供服务，不包括用户。

首先，我们必须定义一个名为 Post 的新类型。 Post 类型的 id 为 Int，文本值为 String。

对于我们的 GraphQL 服务器，我们需要一个名为 RootQuery 的类型。 RootQuery 类型包装了客户端可以运行的所有查询。 它可以是请求所有帖子、所有用户、仅一个用户的帖子等等。 您可以将其与所有使用通用 REST API 的 GET 请求进行比较。 路径将是 /posts、/users 和 /users/ID/posts，以将 GraphQL API 表示为 REST API。 使用 GraphQL 时，我们只有一个路由，我们将查询作为类似 JSON 的对象发送。

我们将要进行的第一个查询将返回一个包含所有帖子的数组。

如果我们查询所有帖子并希望返回每个用户及其对应的帖子，这将是一个子查询，不会在我们的 RootQuery 类型中表示，而是在 Post 类型本身中表示。 稍后您将看到这是如何完成的。
在类 JSON 模式的末尾，我们将 RootQuery 添加到模式属性中。 这种类型是 Apollo Server 的起点。

稍后，我们将向模式中添加突变键，我们将在其中实现 一个 RootMutation 类型。 它将为用户可以运行的所有操作提供服务。 突变类似于 REST API 的 POST、UPDATE、PATCH 和 DELETE 请求。

在文件末尾，我们将模式导出为数组。 如果我们愿意，我们可以将其他模式推送到这个数组以合并它们。
这里缺少的最后一件事是我们的解析器的实现。

### 实现 GraphQL 解析器
现在模式已经准备好了，我们需要匹配的解析器函数。 在graphql文件夹中创建resolvers.js文件，如下：
```

```

解析器对象将所有类型作为属性保存。 在这里，我们设置了 RootQuery，以与我们在模式中相同的方式保存帖子查询。 解析器对象必须与模式相同，但要递归合并。 如果要查询子字段，例如帖子的用户，则必须使用 Post 对象扩展解析器对象，该对象包含 RootQuery 旁边的用户函数。

如果我们为所有帖子发送查询，则执行 posts 函数。 在那里，您可以做任何您想做的事情，但您需要返回与架构匹配的内容。 因此，如果您有一组帖子作为 RootQuery 的响应类型，则您不能返回不同的内容，例如仅返回一个帖子对象而不是数组。 在这种情况下，您会收到一个错误。

此外，GraphQL 检查每个属性的数据类型。 如果 id 定义为 Int，则不能返回常规 MongoDB id，因为这些 ID 属于 String 类型。 GraphQL 也会抛出错误。

为了证明一切正常，我们将继续对我们的服务器执行一个真实的 GraphQL 请求。 我们的帖子查询将返回一个空数组，这是对 GraphQL 的正确响应。 我们稍后会回到解析器函数。 您应该能够再次启动服务器，以便我们可以发送演示请求。

###  发送 GraphQL 查询
我们可以使用任何 HTTP 客户端（例如 Postman、Insomnia 或任何您习惯使用的客户端）来测试此查询。 下一节将介绍 HTTP 客户端。 如果您想自己发送以下查询，则可以阅读下一部分并返回此处。
当您将以下 JSON 作为 POST 请求发送到 http://localhost:8000/graphql 时，您可以测试我们的新功能：

operationName 字段不是运行查询所必需的，但它非常适合用于记录目的。

查询对象是我们要执行的查询的类似 JSON 的表示。 在此示例中，我们运行 RootQuery 帖子并请求每个帖子的 id 和文本字段。 我们不需要指定 RootQuery，因为它是我们 GraphQL API 的最高层。

variables 属性可以保存参数，例如我们想要过滤帖子的用户 ID。 如果要使用变量，也需要在查询中按名称定义它们。

对于不习惯 Postman 等工具的开发人员，还可以选择在单独的浏览器选项卡中打开 /graphql 端点。 您将看到一个用于轻松发送查询的 GraphQLi 实例。 在这里，您可以插入查询属性的内容并点击播放按钮。 因为我们设置了 Helmet 来保护我们的应用程序，所以我们需要在开发中停用它。 否则，GraphQLi 实例不会工作。 只需使用花括号中的以下 if 语句将完整的 Helmet 初始化包装在 server/index.js 文件中：

```
 if(process.env.NODE_ENV === 'production')
```

这种短暂的条件仅在环境处于开发阶段时才会激活 Helmet。 现在，您可以使用 GraphQLi 或任何 HTTP 客户端发送请求。
POST 请求的响应与前面的正文结合时应如下所示：

## 后端调试和日志记录

### Logging in Node.js

### Debugging with Postman
