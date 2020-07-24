---
title: "理解 JavaScript Promise"
date: 2020-02-28
tags:
  - javascript
keywords:
  - javascript
  - promise
---

原文最初写在知乎：https://zhuanlan.zhihu.com/p/26523836 

今天迁移过来，审视的同时做些修改。下面是正文 ヾ(^▽^ヾ)

---

## 前言

本文目标：从一脸懵逼，到认识、掌握并使用 Promise ……

[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/) 规范出来之后，Js 家族丰满了许多，箭头函数、类声明、解构赋值等新特性冒了出来，其中**最重要**的，我认为当属 Promise ，它会在你的前端生涯中占很大的份量。不学会Promise，怎么能说自己懂 ES6 了呢？

那么 Promise 到底是个什么鬼？！像我这样的JavaScript小菜，初次看到，着实一脸懵逼。😳

先来看一些学习资料：

- [JavaScript Promise：简介](https://developers.google.com/web/fundamentals/primers/promises#whats-all-the-fuss-about)
- [Promise - Mozilla Developer Network](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [Promise 对象 - ECMAScript 6入门](https://es6.ruanyifeng.com/#docs/promise)

如果上面的资料，你看过后觉得云里雾里，不甚明白，那么不妨再试试我下面给出的总结

## 本文内容

本文从下面几个问题开始，逐步了解 Promise 的**概念**和**用法**，重点是概念啦，用法什么的网上有很多资料参考

- Promise 是什么？
- 有什么用？解决了什么问题？
- 怎么使用 Promise?

上面的问题即：是什么？为什么？怎么用？

最后整理一些习题、示例，以及参考资料。

## 是什么

当我们谈到、看到或者提到 Promise 的时候，回调函数、异步编程、流程控制这些术语，通常会冒出来，还有，取了 Promise （承诺）这样的名字，究竟是何含义。首先都别 care ，没那么复杂。

Promise 说简单了就是一种写代码的方式，并且是用来书写 JavaScript 程序中异步任务的方式，它被称为**链式语法**。后面会再解释一下异步任务。

### 基本用法

首先看下最基本的用法。第一段要认识的代码如下：

```javascript
const p = new Promise((resolve, reject) => {
  // 做些事情
  // ...
  // 然后程序走到某些条件下 resolve
  // 走到另一些些条件下 reject
  // 这里我用简单的 if 语句表达下上面的意思

  if (/* 是你的条件 😏 */) {
    resolve()
  } else {
    reject()
  }

  // over
})

p.then(() => {
  // 如果 p 的状态被 resolve 了，就进入这里
}, () => {
  // 如果 p 的状态被reject，就进入这里
})
```

### 解释一下

- 第一行调用了 Promise 构造函数，结果是会得到一个对象 p 。
- 第二段是调用了对象 p 上的实例方法 `.then`

#### 1. 构造实例

Promise 构造函数可以接受一个函数作为参数，取名叫做 **executor** ，这里传入的 executor 为：

```javascript
(resolve, reject) => {
  // 做些事情
  // ...
  // 然后程序走到某些条件下 resolve
  // 走到另一些些条件下 reject
  // 这里我用简单的 if 语句表达下上面的意思

  if (/* 是你的条件 😏 */) {
    resolve()
  } else {
    reject()
  }

  // over
}
```

调用构造函数时，构造函数会**立即执行** executor，最后会返回实例 p 。

构造函数执行 executor 时，会传入两个函数 **resolve** 和 **reject** 。

在 executor 内部，如果调用 resolve ，会将 p 的状态变成 **fulfilled** ，或者调用 reject ，会将 p 的状态变成 **rejected** ，那么，在调用 resolve 或 reject 之前，p 的状态就被称为 **pending** 。

前面的 fulfilled 和 rejected 又被统称为 **settled** 状态。

#### 2. 调用 .then

调用 .then 可以为实例 p 注册与前面两种状态对应的回调函数

当实例 p 的状态为 fulfilled ，会触发第一个函数执行

当实例 p 的状态为 rejected ，则触发第二个函数执行

### 总结一下

上面就是使用 promise 最基本的模式，即构造出 promise 实例，然后调用 .then 的编写代码方式。

可以看到，这种方式：

1. 将异步过程（放到 exector 函数内）转化成 promise 对象
2. promise 对象包含3种状态
3. 调用实例方法 .then 能注册状态的回调
4. settled 的状态能触发回调

采用这种方式来处理编程中的异步任务，就是在使用 promise ，也是使用它的目的。

所以 promise 就是一种**异步编程模式**。

## 解决的问题

这需要从 Js 中的异步任务说起……

### 异步任务

setTimeout 调用就是 Js 中一个典型的异步任务：

```javascript
setTimeout(() => {
  // 爱干啥干啥
}, 1000)
```

Js 中的异步任务，可以分成两个部分：**发起**部分，和**后续**部分，即首先需要发起一个任务，然后等到任务完成才能触发后续部分。

比如，在实际项目开发中，经常要使用 AJAX 向服务器请求数据，服务器响应后，前端才获取到数据，然后处理数据用于界面展示。这一过程就分成了**发起请求**，和**处理响应**两部分，代码写出来就是下面这样：

```javascript
getData((data) => {
  // 处理 data
})
// ...发起语句后面的代码
```

上面的 `getData` ，对应到实际项目中，通常就是自己封装好的请求方法，内部用到的可能是 jQuery 中的 `$.ajax`，也可能是现代浏览器提供的 `fetch` API，或者其他。

我们应该知道，`getData` 语句被执行后，紧接着就会执行后面的代码，并不会等待，也不关心什么时候处理请求数据的后续部分（处理 data ）。也就是说，这个 `getData` 调用，不会**阻塞**后面任务的执行，所以它被称为**异步**的任务。


### 回调函数

再来看看异步任务的后续部分……

前面两个例子中，都是采用了**回调函数**来表达后续部分。 promise ，则是另一种表达方式。

既然有回调函数，为什么还需要 promise？

当然是因为回调函数方式不好了！不好到被人称为**回调地狱** (\*/ω＼\*)

也许有人并未觉得回调函数方式有什么不好，确实也不能全盘否定。这就好比操作系统有 windows 、mac os 和 linux ，然而大多数人只知道 windows 。习惯了使用 windows ，并未尝试过其他，自然也就不理解另外两种的好处。别人骂 windows 垃圾，极力推荐其他给你，你也可能不想改变使用习惯……

然而系统可能只是自己使用，代码写出来，通常是会给别人看的，尤其是团队协作开发。

#### 回调哪里不好了？

现在看复杂点的情况。假设有多个异步任务，且任务间有前后依赖关系（每一个任务都依赖前一个任务的结果），回调方式写出来的代码就会像下面这样：

```javascript
getData1(data1 => {
  getData2(data1, data2 => {
    getData3(data2, data3 => {
      getData4(data3, data4 => {
        getData5(data4, data5 => {
          // 终于取到data5了
        })
      })
    })
  })
})
```

这种格式的形状，是不是很像金字塔？所以它被称作**回调金字塔**，也被称为回调地狱。

再假设上面的任务流程里，想要改一下执行顺序？这时候就会发现，代码改起来比较麻烦了，需要上下移动嵌套层级。而且，当真实的业务逻辑填进去，代码变得复杂，阅读起来跳上跳下，会让人头大。

#### 改写

如果用 promise 改写一下：

```javascript
// 假设先把 getData 们都改成了返回 promise 对象的函数

// 然后
getData1()
.then(getData2)
.then(getData3)
.then(getData4)
.then(getData5)
.then(data => {
  // 取到最终data了
})
```

这样的链式风格代码，是线性的，符合人们的阅读习惯，表达出来的流程清晰、易读。


### 任务并行

前面展示的多个异步任务前后依赖，也可称为**串行**。另外还有想要多个异步任务**并行执行**的情况，即多个任务同时发起，等到全部执行完毕后，才统一处理全部结果。

用回调方式来写，可采用下面的办法：

```javascript
const tasks = [getData1, getData2, getData3, getData4, getData5]
const datas = []

tasks.forEach(task => {
  task(data => {
    datas.push(data)

    if (datas.length === tasks.length) {
      // datas 里拿到全部的数据了，开始统一处理
    }
  })
})
```

上面使用了数组，结合 forEach 方法来为每个任务传入回调。

如果改用 promise 来实现，代码会是这样：

```javascript
// 假设先要把 getData 们转成了 promise 对象

// 然后
Promise.all([
  getData1,
  getData2,
  getData3,
  getData4,
  getData5
]).then(datas => {
  // 在这里统一处理全部的数据
})
```

可以看到，promise 的链式风格代码更简洁易读

## 如何使用
 
既然 promise 是一种更好的编程方式，那么接下来就要好好了解下它的具体内容和作用了。

前面已经提到一部分内容了，在本小节里会总结完善一下，怎么用还得看官们自己去实践总结。

接下来的内容力求详细，可能会有些啰嗦，建议大家自行写代码验证会容易理解得多

### 3种状态

首先，promise 实例有 3 种状态：

- pending（待定）
- fulfilled（已执行）
- rejected（已拒绝）

fulfilled 和 rejected 也可以说是已成功和已失败，统称为已完成（settled）状态

### resolve 和 reject

promise 的初始状态是 pending ，在 exector （前面有提到过）内调用 resolve 或 reject ，能将 promise 实例的状态对应变成 fulfilled 或 rejected ，只有变成这两种状态，才能触发 .then 后状态对应的回调

### Promise API

promise 的提供的 api 分为构造函数、实例方法和静态方法：

- 1个构造函数： `new Promise`
- 2个实例方法： `.then` 和 `.catch`
- 4个静态方法： `Promise.all` 、 `Promise.race` 、 `Promise.resolve` 和 `Promise.reject`

下面多说几句它们的作用

#### 构造函数

`new Promise` 常用于将一个异步调用，连带相关的逻辑，改造成 promise 对象。先有了 promise 对象，然后才能够以 promise 的方式编程。

比如：

```javascript
// 改造前的 setTimeout
setTimeout(() => {
  console.log('3秒钟到了')
}, 3000)

// promise 大改造
const promiseTimeout = delay => new Promise(resolve => {
  setTimeout(() => {
    resolve()
  }, delay)
})

// 改造后的 promise 版 setTimeout
promiseTimeout(3000)
  .then(() => {
    console.log('3秒钟到了')
  })
```

#### .then 方法

`.then` 用于为 promise 对象的状态注册回调函数，接受两个参数，它返回的仍然是一个 promise 对象，所以后面还可以连接 `.then` ，后面还可以……，这样就形成了**链式调用**。

`.then` 可以注册两个状态回调函数，第一个叫做 onFulfilled , 第二个叫做 onRejected ，它们中的 `return` 会影响到，该 `.then` 返回的 promise 对象的状态，以及状态携带的**数据**，数据会传递给后面的 onFulfilled 或 onRejected。

`return` 可以分为下面几种， onFulfilled 和 onRejected 中的都一样：

1. `return` 一个非 promise 数据，会将 `.then` 返回的 promise 状态 resolve ，返回的 promise 将该数据向后传递
2. 没有 `return` 语句，等同于末尾 `return undefined` ，归结为情形 1 。
3. `return` 一个 promise ，后面的 `.then` 返回的 promise 行为将取决于这个 `return` 的 promise

#### .catch 方法

`.catch` 仅用于为 rejected 状态注册回调函数，被称为 onRejected 。该回调通常是异常情况的回调，可以是自己认为的异常，还包括程序运行错误情况，即如果前面的 Js 语句运行中出错，就会进入该回调，`.then` 中注册的 onRejected 完全一样。返回同.then一样，也是新的 promise 对象。

#### 链式调用

可概括成**就近原则**：当前面的 promise 状态为 settled （被 resolve 或 reject ）后，会触发后面离它最近的 onFulfilled 或 onRejected 回调。

#### 静态方法

如果理解了前面的概念，剩下的也就不难了，自行参考对应的文档即可。这里只做简单说明

- 调用 `Promise.resolve` 会立即返回一个状态为 fulfilled 的 promise 对象，参数会作为数据，传递给后面的对应的状态回调
- `Promise.reject` 与 `Promise.resolve` 同理，区别在于返回的 promise 对象状态为 rejected
- `Promise.all` 则用来并行处理多个 promise ，前面已经提到过
- `Promise.race` 也是用来并行处理多个 promise ，与 `Promise.all` 的区别在于，它表示有一个 promise 状态变成 settled 了，就触发后续状态回调

## 未完待续

ES7 给出了 promise 的一些新的 API ……

## 有趣的习题

网上找到一个有意思的 promise 面试题：[实现红绿灯交替亮灯](https://www.cnblogs.com/dojo-lzz/p/5495671.html)

贴一下我的解答：

```javascript
// 思路：先用promise控制三种灯的执行顺序，然后用递归实现循环亮灯

const red = () => {
  console.log('red');
}

const green = () => {
  console.log('green');
}

const yellow = () => {
  console.log('yellow');
}

const light = (fn, timer) => new Promise(resolve => {
  setTimeout(() => {
    fn()
    resolve()
  }, timer)
})

// times为交替次数
const start = (times) => {
  if (!times) {
    return
  }

  times--
  Promise.resolve()
    .then(() => light(red, 3000))
    .then(() => light(green, 1000))
    .then(() => light(yellow, 2000))
    .then(() => start(times))
}

start(3)
```
