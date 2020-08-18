---
title: npm使用指南
date: 2020/7/25
permalink: /npm-user-guide/
---

npm全称Node Package Manager，顾名思义，它是基于Node.js开发的包管理工具，当你的Javascript项目需要安装第三方依赖包时，可通过简单的npm install命令来下载并安装代码到本地，最重要是下载的所有包的依赖包也都会逐一自动搜索下载安装并管理，十分方便。

## 安装npm的CLI

最简单的安装方法是从Node.js官网下载[Node.js安装包](https://nodejs.org/zh-cn/download/)，npm的命令行工具（CLI）会随着Node.js的安装附带。

从下载页面中选取自己对应的OS平台及需要的版本（尽量用LTS版），例如我打开官网看到的当前推荐的LTS版本号是**12.18.3** (包含 npm 6.14.6)。

下载并安装后，可以打开命令行终端，输入命令查看版本号：`npm -v`

一般而言，我们除了会调用别人的模块之外，还可能会开发自己的模块或命令行工具并公开出去给自己或别人下载使用，所以下面会这从两个方面来进行介绍。

## 安装第三方npm包

### 初始化项目

在安装第三方包之前，我们可以将自己现有的项目（目录）初始化为一个npm包，运行`npm init`命令，会输出如下交互式的命令，只需要按照自己的实际项目信息填写并按Enter键逐步完成即可：

```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (blank) testnpm
version: (1.0.0)
description: test npm cli
entry point: (index.js)
test command:
git repository:
keywords:
author: haobrother
license: (ISC)
About to write to J:\fsd\test\blank\package.json:

{
  "name": "testnpm",
  "version": "1.0.0",
  "description": "test npm cli",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "haobrother",
  "license": "ISC"
}


Is this OK? (yes) yes
```

上面命令运行完毕后，可以看到项目目录下会自动生成一个package.json文件，它代表项目目录从此就变成了一个npm包！

#### package.json文件

我们经常会在别人的项目或者包里看到package.json文件，它意味着这是一个基于npm开发Javascript项目，简单而言它是一个npm项目配置文件，使用编辑器打开这个文件我们可以看到很多字段，包括项目的名字(name)、版本（version）、描述（description）、作者（author）等等，有两个重点字段**dependencies**和**devDependencies**用来记录本项目对应的依赖包及其版本，所有的这些信息都可以根据自己需要编辑修改，更多相关信息可以参考[npm官方文档](https://docs.npmjs.com/configuring-npm/package-json.html)。

### 安装npm包

```
npm install <包名>
```

只要这么一条命令，就能简单地把一个第三方的npm依赖包安装到当前目录的node_modules目录（该目录专门用来存放依赖包），之后就可以在你的项目代码里面使用Javascript模块化技术引入刚安装的包，如果是使用CommonJS的话使用require()，而ES6 Modules则可以用import引入。

如果你从网上下载了一个别人的项目到本地，通常是不会自带node_modules目录的，这时候构建这个项目最快的方式是直接无参数运行`npm install`，它会自动安装所有在package.json中记录的依赖包。

#### 本地安装VS全局安装

npm install <包名>默认是将依赖包安装到本地，所谓的本地就是当前目录下的node_modules目录，其实还可以全局安装npm包，只需要加上-g参数即可：

```
npm instal -g <包名>
```

这样会把npm包安装在node的安装目录下。

总的来说，对于项目特定的依赖包使用本地安装（不带-g参数），而对于一些命令行工具包，例如vue-cli和create-react-app等，可以使用全局安装。

#### dependencies和devDependencies

前面提到package.json文件的两个重要字段——**dependencies**和**devDependencies**，它们分别用来记录当前项目包的生产环境依赖和开发环境依赖。

对于开发环境依赖，可以使用-D或者--save-dev，这样依赖包会记录在**devDependencies**，例如；

```
npm install -D webpack
```

对于生产环境依赖，（默认情况）使用-S或者--save，这样依赖包会记录在**dependencies**，例如：

```
npm install -S axios 等同于 npm install axios
```



对于npm包来说，有两种环境：

1. 一种是开发环境，这种环境下需要向node_modules下的模块和包获取相关资源进行开发，所以对当前的包来说是开发环境；
2. 另一种是生产环境（正式运行环境），相对于开发环境来说，当这个包作为依赖被其他包所引用时，那么对这个依赖包来说就是在生产环境；

像webpack、babel等这种只在开发时中才用到的依赖包，我们只需把它放在devDependencies，而其他像vue、react、bootstrap这种则是生产环境所必须的，应该放在dependencies。

举一个简单的例子，假设有以下两个模块： 

---

**模块A** 

\- devDependencies （开发环境依赖）

  模块B 

\- dependencies （生产环境依赖）

  模块C 

---

**模块D** 

\- devDependencies （开发环境依赖）

  模块E 

\- dependencies （生产环境依赖）

  模块A 

---

对于这两个模块有两种使用场景：

1. 当我们想在自己的项目包中引用模块D的代码，那么只需要运行`npm install D`，这时候下载到本地的模块为D、A和C。

   解释：对于当前的项目来说，模块D属于一个生产环境依赖，所以我只需要下载它的dependencies，即这里的模块A以及模块A的dependencies（一层层传递下去），但不包含E和B。

2. 如果我们想研究模块D这个项目，并下载了它的源码到本地，并在其目录下运行`npm install`，那么下载到该目录的模块为A、C和E。

   解释：对于模块D这个项目而言，运行`npm install`就是从D这个项目中下载所有依赖（包括devDependencies和dependencies )，但对下一级包（在这里是模块A）的依赖，只需要下载其生产环境依赖。

### 更新/删除包

查看当前项目中所有或特定的依赖包是否过时：

```
npm outdated 或 npm outdated <包名>
```

更新当前项目中所有或特定的依赖包：

```
npm update 或 npm update <包名>
```

删除特定的依赖包：

```
npm uninstall <包名>
```

## 创建并发布自己的npm包

### 注册npm账号

首先到npm官网（https://www.npmjs.com/）注册一个npm账号，主要是填写账号、密码和邮箱，注册完成后可以登入npm，不过还需要到注册邮箱下验证一下账号才行。

### 登录npm账号

回到自己的终端，输入以下命令进行登录：

```
npm login
```

然后输入刚才注册的账号密码和邮箱即可。

以后每当不确定自己登录的npm账号是哪个时，可以输入`npm whoami`来看下。

注意，如果你的电脑正在使用非官方源，那么需要切换一下：

```
npm config set registry  https://registry.npmjs.org/
```

这样才能保证终端登录到官方npm源，下面就可以正常发布你的包了。

### 发布npm包

打开你的终端并进入你打算发布的包所在目录，然后输入下面命令发布到npm：

```
npm publish
```

注意一点的是，你的包需要保证不与npm目前存在的任一个包名重复，所以当报类似“npm ERR! 403 Forbidden - PUT https://registry.npmjs.org/testnpm - Yoe permission to publish "testnpm". Are you logged in as the correct”的错误时，需要到package.json文件中修改你的包名（name），然后再输入发布命令。

接着可以到npm搜索一下自己的包（我的包名是hao-testnpm），结果没让人失望，那我们可以把自己的包下载下来使用了。

例如下面是我发布的一个模块index.js（这个文件名需要与你的package.json文件中main字段一致）：

```
function max(a, b) {
	return a > b ? a : b;
}

module.exports = {
	max,
};
```

为了使用这个刚发布的包，可以在使用的项目中输入命令：

```
npm install hao-testnpm
```

接下来可以用以下方法使用这个模块：

```
const max = require('hao-testnpm').max;
console.log(max(2,3));
```

### 更新npm包

#### 修改版本号

发布自己包之后，我们可能会有更新内容，那么除了需要更新自己的模块代码之外，还需要更新package.json的版本号，除了手动修改之外的方法外，更方便的方式是：

```
npm version [patch | minor | major]  
```

下面简单说明一下npm version命令的三个参数。

patch：小改动，只修改最后一位版本号，例如1.0.0修改为1.0.1

minor：功能性修改，不影响现有功能，修改第二位版本号，例如1.0.0修改为1.1.0

major： 破坏性更改，修改第一位版本号，例如1.0.0修改为2.0.0

#### 再发布

```
npm publish
```

#### 更新项目中的依赖包

在发布之后，如果打算再更新自己的包，可以输入以下命令：

```
npm update <包名>
```

或者

```
npm install <包名>@<版本号>
```

例如我这里更新到1.1.0版本，那么可以使用`npm install hao-testnpm@1.1.0`来更新依赖包。