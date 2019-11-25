---
title: async&promise
date: 2019-11-15 13:25:37
tags:
---

## js 中多个异步操作的串行并行（await/async 和 Promise）

> async/await 的目的是简化使用多个 promise 时的同步行为，并对一组 Promises 执行某些操作。正如 Promises 类似于结构化回调，async/await 更像结合了 generators 和 promises。

##### 代码实例

```js
// 定义两个时长不同的异步操作函数
var slow = function() {
  return new Promise(resolve => {
    setTimeout(function() {
      resolve("slow");
    }, 2000);
  });
};
var fast = function() {
  return new Promise(resolve => {
    setTimeout(function() {
      resolve("fast");
    }, 1000);
  });
};
```

1. 串行多个异步操作

```js
var sequential = async function() {
  //1. 立即执行到这里
  const slowLog = await slow();
  console.log(slowLog); //2. 1之后2秒到这里
  const fastLog = await fast();
  console.log(fastLog); //3. 1之后3秒到这里
};
```

2. 并行多个异步操作

- 同时启动，全部完成后输出

```js
// await/async 并行异步，结果同一时间点输出log
var concurrentStart = async function() {
  const slowLog = slow(); // 异步操作立即启动
  const fastLog = fast(); // 异步操作立即启动
  // 1. 立即执行到这里
  console.log(await slowLog); // 2. 1之后2秒到这里
  console.log(await fastLog); // 3. 2之后立即执行到这里，因为fast已经有resolve结果
};

// promise 并行异步，结果同一时间点输出log
var concurrentPromise = function() {
  //1. 多个异步操作同时立即启动
  return Promise.all([slow(), fast()]).then(messages => {
    console.log(messages[0]); // 1之后2秒到这里
    console.log(messages[1]); // 1之后2秒到这里
  });
};
```

- 同时启动，各自完成后输出

```js
var parallel = async function() {
  // await/async 并行异步
  await Promise.all([
    (async () => console.log(await slow()))(), //2秒后输出
    (async () => console.log(await fast()))() //1秒后输出
  ]);
};

// promise 并行异步
var parallelPromise = function() {
  slow().then(message => console.log(message)); //2秒后输出
  fast().then(message => console.log(message)); //1秒后输出
};
```

---

<table><td bgcolor=#F56C6C  >async函数有可能错误地忽略错误。以parallel异步函数为例。 如果它没有等待await（或返回）Promise.all([])调用的结果，则不会传播任何错误。
虽然parallelPromise函数示例看起来很简单，但它根本不会处理错误！ 这样做需要一个类似于return Promise.all([])处理方式。</td></table>
