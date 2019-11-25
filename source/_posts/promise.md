---
title: promise
---

## js 标准内置对象 promise

> Promise 对象用于表示一个异步操作的最终完成 (或失败), 及其结果值.

##### 创建 promise 语法

```js
new Promise( function(resolve, reject) {...} /* executor */  );
```

<font size=2>**executor** 是带有 resolve 和 reject 两个参数的函数 。Promise 构造函数执行时立即调用 executor 函数， resolve 和 reject 两个函数作为参数传递给 executor。resolve 和 reject 函数被调用时，分别将 promise 的状态改为 fulfilled（完成）或 rejected（失败）。executor 内部通常会执行异步操作，异步操作执行完毕，要么调用 resolve 函数来将 promise 状态改成 fulfilled，要么调用 reject 函数将 promise 的状态改为 rejected。如果在 executor 函数中抛出一个错误，那么该 promise 状态为 rejected。executor 函数的返回值被忽略</font>

##### Promise 状态:

- pending: 初始状态，既不是成功，也不是失败状态。
- fulfilled: 意味着操作成功完成。
- rejected: 意味着操作失败

> 注意： 如果一个 promise 对象处在 fulfilled 或 rejected 状态而不是 pending 状态，那么它也可以被称为 settled 状态。你可能也会听到一个术语 resolved ，它表示 promise 对象处于 settled 状态。

##### 书写实例

```js
const myPromise = new Promise((resolve, reject) => {
  // 这里一般执行异步操作
  reslove(someValue); //fulfilled
  //？或
  reject("failure reason"); //rejected
});
```
