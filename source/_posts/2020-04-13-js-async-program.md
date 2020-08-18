---
title: JS异步编程
date: 2020/4/13
permalink: /js-async-program/
---

自从工作以来我发现一直离不开的一个技术是JS异步编程，回调函数是其中最为经典的一种实现方式，这已经在许多经典Web API（XHR等）中使用过，而在使用Promise和async/await过程中感觉自己对里面的一些概念还是模糊不清，导致使用起来不够得心应手，写出来的代码更是凌乱，现在趁有空赶紧从网络上搜集相关资料学习一番，然后记录下自己的理解。

## 同步与异步

要理解异步编程，首先得搞清楚它的对立面——**同步**，这是我们写得最多的一种代码，利用这种方式写代码非常符合直觉，一步接着一步地完成计算、操作，最后得出结果。但不可否认的是，总会有一些代码不会那么快完成，或者它们必须达到某种条件才能够执行，如果这类代码运行的话，由于长时间还没有把代码控制权交还给浏览器会导致页面“假死”（什么都操作不了），也就是**阻塞**：

```js
for(let i = 0; i < 1000000; i++) {
    ctx.fillStyle = 'rgba(0,0,255, 0.2)';
    ctx.beginPath();
    ctx.arc(random(0, canvas.width), random(0, canvas.height), 10, degToRad(0), degToRad(360), false);
    ctx.fill()
}

alert('按下！')
```

上面的代码的for循环是一个耗时的绘制操作，它会导致后面的alert提示中的`按下！`不会立即弹出，而是会等待前面的耗时代码执行完成之后才会弹出，因为这时两者是同步执行的。

### JS是单线程

为什么会这样呢？明明我的PC使用的是多核CPU，但这些代码却不能分别同时执行？

因为Javascript设计上就单线程，即使有多个CPU内核，它也只能运行在单一线程上执行代码，这个线程成为主线程，现代Javascript为了解决这个问题提供了两种解决方案：异步编程和Web workers。

Web workers一般用于密集运算，但不能涉及DOM操作，所以不能直接更新UI是它的主要限制，其次worker里面的代码也是同步的，如果有一个函数依赖于几个在它之前运行的过程的结果，那么这样会出现问题：

```html
主线程: 任务A --> 任务B
```

假设任务A正在从服务器获取图片，然后任务B需要对这个图片进行某种操作，如果这是在主线程中执行的同步代码的话肯定会出错，因为任务A不是一个立即就能完成的操作（请求服务器并响应处理需要一定的时间），而这个时候任务B立即进行执行会以为操作一个空对象而失败。

为了解决这类问题，我们需要另外一种方式——异步代码。没错，**异步代码就是用来解决JS单线程执行同步代码时会遇到阻塞的问题**，对于这类情况，我们可以使用异步编程！

使用异步编程可以让我们的某些任务在之后某个时间点才执行：

```html
主线程:     任务A                    任务B
Promise:      |_______ 异步操作 ______|
```



## 编写异步JS代码

总的来说有两类异步代码编写方式，分别为**Callback**（回调函数）和**Promise**。

### Callback

对于回调函数，大多数人都很熟悉，因为无论是老式异步请求XHR中onload属性，还是浏览器DOM事件绑定方法addEventListener都是使用回调函数来实现异步代码执行：

```js
btn.addEventListener('click', () => {
  alert('按下！');
});
```

这里的箭头函数就是一个**异步回调函数**，值得注意的是，不是所有回调函数都是异步的，因为有这些例外：Array.prototype.forEach()、Array.prototype.map()等，它们的参数都是传入一个同步执行的回调函数。

对于异步回调，除了原有的Web API提供之外，也可以自己封装一个：

```js
function loadAsset(url, type, callback) {
  let xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.responseType = type;

  xhr.onload = function() {
    callback(xhr.response);
  };

  xhr.send();
}

function displayImage(blob) {
  let objectURL = URL.createObjectURL(blob);

  let image = document.createElement('img');
  image.src = objectURL;
  document.body.appendChild(image);
}

loadAsset('coffee.jpg', 'blob', displayImage);
```

这里的loadAsset就是一个封装了XHR请求图片的异步回调API，而`displayImage`就是异步回调函数，它在等待图片响应并返回到客户端后执行处理。

### Promise

Promise是现在JS中的主流异步编程方式。根据字面意思，Promise定义了一个承诺，它承诺要做一件事，而这件事不会立即就完成，但做完这件事之后会通知你，结果可能是成功或失败。

语法原型如下：

```js
new Promise( function(resolve, reject) {...} /* executor */  );
```

可以看到传给Promise的是一个叫executor的函数，也叫执行函数，这个函数就是承诺会做的事，但它可能不会立即就完成，但是会在完成后通知你，怎么通知呢？通过执行函数的两个参数resolve和reject来通知，而这两个都是函数，resolve用来在承诺完成后将承诺状态修改为fulfilled（完成），而reject用来在承诺失败后拒绝并给出reason（原因），以便后续进行不同的处理。任何一个承诺都可以通过后接一个then方法，以传达在其fulfilled（完成）或者rejected（拒绝）后需要进行什么样的处理。then方法有两个参数，一个是完成后执行的函数onFulfillment，另一个是拒绝后执行的onRejection，两者都有一个参数用来分别接收来自resolve和reject的参数。另外，catch方法也可以代替onRejection来捕获并处理Promise上的错误。

一个Promise的处理流程：

![img](https://mdn.mozillademos.org/files/8633/promises.png)

从上图可以看到，一开始Promise的状态为pendding，等执行函数处理完后可能变为fulfilled（settled）或者rejected。Promise可以随时后接then来对结果进行后续处理，还可以在then方法后面继续接then或catch方法，因为then/catch方法返回的也是一个Promise，所以Promise可以进行链式调用，并使得异步操作一个接一个地进行；另外catch除了能够捕获整条链前面的错误，也可以在处理完错误之后接then继续执行异步任务。

下面是一个Promise示例：

```js
let myFirstPromise = new Promise(function(resolve, reject){
    //当异步代码执行成功时，我们才会调用resolve(...), 当异步代码失败时就会调用reject(...)
    //在本例中，我们使用setTimeout(...)来模拟异步代码，实际编码时可能是XHR请求或是HTML5的一些API方法.
    setTimeout(function(){
        resolve("成功!"); //代码正常执行！
    }, 250);
});

myFirstPromise.then(function(successMessage){
    //successMessage的值是上面调用resolve(...)方法传入的值.
    //successMessage参数不一定非要是字符串类型，这里只是举个例子
    console.log("Yay! " + successMessage);
});
```

上面的方法是当我们需要自己生成Promise时使用的方法，而大多时候使用WebAPI或者第三方库的异步函数本身就返回Promise，那么可以像这样：

```js
doSomething().then(function(result) {
  return doSomethingElse(result);
})
.then(function(newResult) {
  return doThirdThing(newResult);
})
.then(function(finalResult) {
  console.log('最终结果: ' + finalResult);
})
.catch(failureCallback);
```

另外，Promise还有一些实用的静态方法：

- `Promise.all`方法，可以同时执行多个承诺并等他们都完成后返回它们的结果数组，或者任意一个承诺失败会立即拒绝它并返回reason。
- `Promise.race`，可以同时执行执行多个承诺并只要有一个完成后返回它的结果，或者任意一个承诺失败会立即拒绝它并返回reason。
- `Promise.resolve(value)`，根据value值来处理返回值，如果value是一个Promise，则直接返回该Promise；如果value是一个含有thenable对象（含有then方法），则返回一个fulfilled的Promise对象，并且其then方法的参数为这个thenable对象的then返回值；所有其他value值直接返回一个fulfilled的Promise对象且后接的then方法中的onFulfilled参数为该value值。
- `Promise.reject(reason)`，返回一个状态为被拒绝的Promise，并将给定的失败原因reason传递给对应的处理方法（catch）

### async/await

对于熟悉了Promise的童鞋一定会认为链式调用已经够爽了，但是其实还有一种更加舒服的写法，那就是aysnc/await，async和await关键字属于ES7的新语法，可以让你像编写同步代码那样编写异步代码！

首先要将一个普通函数声明为异步函数，在function前添加async：

```js
async function hello() { return "Hello" };
hello();
```

这里的hello函数调用返回了一个Promise，而不是"Hello"字符串，也就是说本质上的async函数就是封装了一个Promise。

只要一个函数声明为async函数那么就可以使用await来用“暂停”代码！现在，hello函数里面可以使用await关键字来处理任意的异步操作，也就是将await放在返回Promise的函数前面，当代码执行到该行就会暂停下来等待，直到该Promise的异步操作完成再继续执行下一行：

```js
async function myFetch() {
  let response = await fetch('coffee.jpg');
  let myBlob = await response.blob();

  let objectURL = URL.createObjectURL(myBlob);
  let image = document.createElement('img');
  image.src = objectURL;
  document.body.appendChild(image);
}

myFetch()
.catch(e => {
  console.log('出错了: ' + e.message);
});
```

注意的是，只有声明为async的异步函数里面才可以使用await关键字，普通函数不能使用！

#### 错误处理

捕获错误的话可以在async函数里使用try...catch语句。

```js
async function myFetch() {
  try {
    let response = await fetch('coffee.jpg');
    let myBlob = await response.blob();

    let objectURL = URL.createObjectURL(myBlob);
    let image = document.createElement('img');
    image.src = objectURL;
    document.body.appendChild(image);
  } catch(e) {
    console.log(e);
  }
}

myFetch();
```

但因为异步函数的本质也是返回一个Promise，可以直接在其后接catch块：

```js
async function myFetch() {
  let response = await fetch('coffee.jpg');
  return await response.blob();
}

myFetch().then((blob) => {
  let objectURL = URL.createObjectURL(blob);
  let image = document.createElement('img');
  image.src = objectURL;
  document.body.appendChild(image);
})
.catch((e) =>
  console.log(e)
);
```

#### async/await的缺陷

大量await可能会阻塞代码：

```js
function timeoutPromise(interval) {
  return new Promise((resolve, reject) => {
    setTimeout(function(){
      resolve("完成");
    }, interval);
  });
};

async function timeTest() {
  await timeoutPromise(3000);
  await timeoutPromise(3000);
  await timeoutPromise(3000);
}

var startTime = Date.now();
timeTest().then(() => {
  let finishTime = Date.now();
  let timeTaken = finishTime - startTime;
  alert("耗时（ms）: " + timeTaken);
})
```

上面代码会弹出“耗时（ms）：9000”（也许会有误差几毫秒）

而你实际上可能想要同时开始多个异步操作，有一个办法可以做到：定义多个变量来存储多个异步Promise：

```js
function timeoutPromise(interval) {
  return new Promise((resolve, reject) => {
    setTimeout(function(){
      resolve("完成");
    }, interval);
  });
};

async function timeTest() {
  const timeoutPromise1 = timeoutPromise(3000);
  const timeoutPromise2 = timeoutPromise(3000);
  const timeoutPromise3 = timeoutPromise(3000);

  await timeoutPromise1;
  await timeoutPromise2;
  await timeoutPromise3;
}
var startTime = Date.now();
timeTest().then(() => {
  let finishTime = Date.now();
  let timeTaken = finishTime - startTime;
  alert("耗时（ms）: " + timeTaken);
})
```

经过改良，上面代码会弹出“耗时（ms）：3000”（也许会有误差几毫秒）

另外，不能在非async函数内或者代码的顶级上下文使用await关键字，因此需要创建额外的函数，略有麻烦，但是一般都是值得的。