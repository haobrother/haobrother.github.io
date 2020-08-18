---
title: Node.js入门
date: 2020/8/8
permalink: /nodejs-get-started/
---

Node.js是一个开源跨平台的Javascript运行时环境，基于Chrome浏览器内置的Javascript引擎——V8打造而成，它继承了Javascript单线程运行的原理，具有**非阻塞异步I/O**的特点，因此特别适合用来开发高性能的可伸缩性强的系统。

### 什么是Node.js

简单点说，Node.js就是一个运行JS代码的环境，毕竟它把Javascript解释引擎从浏览器上扒下来，然后添加一些其他东西（libuv/llhttp/openssl等）改造成了自己的样子，所以这里要跟浏览器的JS环境区分开来（后面会说）。

Node.js的诞生催生了很多前端开发工具的流行，比如用得最多的npm（最大的代码包仓库），Webpack（前端打包工具），还有vue-cli、create-react-app（各种工程化的CLI工具）等等。

不过最具革命性的一点是，它使得前端开发人员可以很容易地利用JS语言开发后端应用，例如Web服务器。

另外，在语法层面上来说，我觉得Node.js最大的特点就是**异步**，与传统语言（C/C++/Java/C#等）相比，编写Node.js代码根本不用考虑多线程的问题，只要执行到一些耗时的I/O操作（请求网络，读写文件等），它都会立即返回并继续执行后面的代码，然后根据事件循环的机制等到完成这些I/O操作后再回来执行的后续代码。这种单线程异步执行的特性使得Node.js非常适合执行高并发的任务，而无需担心多线程引起的各种问题，并且能够充分利用CPU的性能。

同步与异步执行代码的对比：

```
// 同步
const fs = require('fs');
const data = fs.readFileSync('/file.md'); // 阻塞
// 下面代码需要等待前面的文件读取完毕并返回后才能继续执行

/////////////////////////

// 异步
const fs = require('fs');
fs.readFile('/file.md', (err, data) => {   // 非阻塞
  if (err) throw err;
});
// 调用fs.readFile之后，不需要等文件读取完毕，继续执行下面代码
```

Node.js作为使用Javascript的运行环境，与浏览器的不同点：

#### 浏览器

- 与DOM交互
- Web API，例如Cookie操作等
- document，window对象
- 无法访问本地文件系统
- 无法确定用户使用的环境（浏览器版本），也就是说无法确定用户浏览器对于最新的ES标准支持的力度，而对于最新的ES标准需要使用babel
- 模块化使用ES6 Module，语法关键词是import

#### Node.js

- 没有DOM概念
- 不能访问Web API
- 没有document，window对象
- 提供了fs模块访问本地文件系统的模块
- 可以根据自己开发的版本确定部署机运行的Node.js版本，这意味着可以随时使用ES6-7-8-9之类的最新语法标准进行开发（而不需要babel这类外来拓展）
- 模块化使用CommonJS，语法关键词是require()

### 安装方法

一般来说，有两种安装Node.js的方法：

- 从[Node.js官网](https://nodejs.org/zh-cn/download/)下载安装包后直接安装，一般选用LTS版本

- 利用nvm工具下载安装，好处是可同时安装多个不同版本的Node.js并随时切换使用。

  个人推荐使用[nvm](https://github.com/nvm-sh/nvm)，这样可以更加灵活地根据项目需求来运行不同版本的Node.js。windows则需要安装[nvm-windows](https://github.com/coreybutler/nvm-windows)。

  安装完毕后，运行以下命令来查看安装是否成功：

  ```
  node -v
  ```

  上面的命令会打印出当前安装（使用）的Node.js版本号。

### 使用场景

对于前端开发人员来说，Node.js应该是必备的开发环境，但是大多数都是使用基于Node.js开发的CLI工具，例如大家都在使用的npm、webpack、vue-cli、craete-react-app等。然而实际上，Node.js对于前端人员来说，真正革命性的地方是服务端开发能力。

#### web服务器

Node.js的诞生使得前端开发人员利用JS开发服务端应用成为可能，而且在某些场合还特别适用，例如高并发要求的游戏、实时聊天室等。

这里作为入门演示，只讲解如何使用Node.js搭建一个最基本的HTTP Web服务器：

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

可以看到，首先引入了一个Node.js内置的http模块，然后通过createServer方法场景一个服务器实例，这个方法通过传入一个带有两个参数（分别是请求实例req和响应实例res）的回调函数来处理用户请求和响应，最后通过listen方法监听服务器指定端口和域名来启动服务器。

通过将上面的代码保存到文件app.js，然后运行以下命令：

```
node app.js
```

然后打开浏览器，在地址栏输入127.0.0.1:3000来访问刚创建的服务，可以看到响应的“Hello World”。

显然，这个示例不足以表现Node.js的强大，除了http模块之外，Node.js还有非常多有用的内置模块，例如文件管理的fs模块，路径处理的path模块，操作系统信息的os模块、事件处理的events模块等等，更多的API说明可以参考[Node.js官方文档](https://nodejs.org/zh-cn/docs/)。

另外，对于Node.js web服务器的入门实践，我推荐一个简短精悍的在线教程——[Node入门](https://www.nodebeginner.org/index-zh-cn.html)。

#### CLI工具

CLI工具就是命令行接口工具（command line interface），例如npm、webpack以及vue-cli,都是使用Node.js开发的CLI工具。通过npm安装的node.js包，除了代码模块之外，还可能是一个可执行的CLI工具，那么我们是否也可以利用Node.js开发自己的CLI工具呢？答案当然是可以的。

从上面可以得知，运行一个Node.js应用的时候使用node命令：

```
node app.js
```

而使用npm下载一个模块的时候使用npm命令：

```
npm install <包名>
```

那么，如果我想开发一个自己的Node.js CLI工具，命令的名字就叫getmax，这个命令用来获取两个数字中的最大值，类似这样：

```
getmax 2 3
3  // 通过getmax对后续的两个参数的处理，输出最大值
```

那么该怎么做呢？

##### 初始化CLI项目

首先，创建一个Node.js CLI项目，这其实跟一般的npm项目包初始化一样：

```
npm init
```

输入上面命令后，可以按照自己需求配置信息，也可以一路回车，然后后面再根据自己需求来配置package.json的信息。

然后在当前目录编辑主文件index.js：

```
console.log('自己的CLI工具！');
```

没错，暂时就这么简单！保存后，运行`node index.js`会输出“自己的CLI工具”。

等等，这个好像跟我们想要的有点不同，如果我们写完主文件的逻辑，发布出去后别人安装我们的包也要这样运行`node index.js`，岂不是很麻烦？我想要直接运行getmax命令，需要怎么做？

现在要先给主文件的头部添加一句 **#!/usr/bin/env node**：

```
#!/usr/bin/env node
console.log('自己的CLI工具！');
```

这一句的意思是在node环境中运行我们的当前文件，保存文件后我们可以这样运行主文件：

```
./index.js
```

这样主文件index.js本身就具备了可执行的能力了！

那么如何通过npm配置我们的CLI工具呢？，从而将getmax命令与index.js主文件关联在一起呢？很简单，在package.json中添加bin字段，并填入主文件路径：

```
{
  "name": "getmax",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "bin": {"getmax": "./index.js"},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

然后安装当前CLI工具的npm包到自己的全局目录，运行命令`npm install -g`，然后我们试试运行：

```
getmax
自己的CLI工具！
```

##### 命令行参数

接下来完成CLI工具的逻辑编写，一般命令行都有参数，我们可以通过Node.js提供的**process.agrv**来访问所有的命令行参数，这是一个数组，其中前两项固定是node命令和自定义命令的主文件路径，所以有效的命令参数实际上是从第三项开始，修改index.js为如下：

```
#!/usr/bin/env node
const first = process.argv[2];
const second = process.argv[3];
function getMax(a, b) {
	return a > b ? a : b;
}
console.log(getMax(first, second));
```

保存文件后，运行命令：

```
getmax 3 6
6
```

这样便完成了一个Node.js CLI工具的制作。

### 社区资源与框架

Node.js属于一个底层平台，而在社区中已经从这个极具潜力的平台中孕育出许多十分优秀的应用框架或工具：

- [**AdonisJs**](https://adonisjs.com/): 高度关注开发人员工效学、稳定性和信心的全栈框架。Adonis是最快的Node.js web框架之一。
- [**Express**](https://expressjs.com/): 它提供了创建web服务器的最简单而强大的方法之一。它最简单的方法，不受限制的，专注于服务器的核心特性是它成功的关键。
- [**Fastify**](https://fastify.io/): 一个专注于用最少的开销和强大的插件架构提供最好的开发者体验的web框架。Fastify是最快的Node.js web框架之一。
- [**hapi**](https://hapijs.com): 用于构建应用程序和服务的丰富框架，使开发人员能够专注于编写可重用的应用程序逻辑，而不是花时间构建基础设施。
- [**koa**](http://koajs.com/): 它是由Express背后的同一个团队构建的，目标是更简单、更小，建立在多年的知识基础上。新项目产生于创建不兼容的变更而又不破坏现有社区的需要。
- [**Loopback.io**](https://loopback.io/): 使构建复杂的现代应用程序变得容易。
- [**Meteor**](https://meteor.com): 一个非常强大的全栈框架，用同构的方法来构建JavaScript应用程序，在客户端和服务器上共享代码。曾经是提供一切的现成工具，现在集成了[React](https://reactjs.org/)、[Vue](https://vuejs.org/)和[Angular](https://angular.io)前端库，也可以用来创建移动端App。
- [**Micro**](https://github.com/zeit/micro): 它提供了一个非常轻量级的服务器来创建异步HTTP微服务。
- [**NestJS**](https://nestjs.com/): 一个基于TypeScript的渐进Node.js框架，用于构建企业级高效、可靠和可扩展的服务器端应用程序。
- [**Next.js**](https://nextjs.org/): 用于服务端渲染的[React](https://reactjs.org/)应用框架。
- [**Nx**](https://nx.dev/): 一个使用NestJS、Express、[React](https://reactjs.org/)、[Angular](https://angular.io)等开发全栈单orepo的工具包!Nx帮助您将开发从一个团队构建一个应用程序扩展到多个团队协作多个应用程序!
- [**Socket.io**](https://socket.io/): 一个建立网络应用的实时通信引擎。