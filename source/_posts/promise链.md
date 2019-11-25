---
title: promise链
date: 2019-11-15 15:40:37
tags:
---

## promise 链

> 返回 Promise 的 API 将会产生一个 promise 链

##### 在请求 promise 之后，.then 或.catch 情况下分别继续进行请求 promise 操作

```js
function api(url) {
  //返回一个请求数据操作
  return downloadData(url) // 返回一个 promise 对象
    .catch(e => {
      // 异常时发出一个请求备用数据
      return downloadFallbackData(url); // 返回一个 promise 对象
    })
    .then(v => {
      // 正常时发出一个新请求并传递接收的数据
      return processDataInRes(v); // 返回一个 promise 对象
    });
}
```

##### 使用 async 重写，如下

```js
async function api(url) {
  let v;
  //标记尝试语句块
  try {
    v = await downloadData(url); //发出请求数据操作
  } catch (e) {
    v = await downloadFallbackData(url); //异常时发出一个请求备用数据
  }
  return processDataInRes(v); //// 正常时发出一个新请求并传递接收的数据
}
```

<table><td bgcolor=#F56C6C  >这里return 语句中没有 await 操作符，因为 async function 的返回值被隐式传递给 Promise.resolve</td></table>

##### 使用 return await 重写,并比较 return await 和 return 的区别

```js
async function api(url) {
  let v;
  try {
    v = await downloadData(url);
  } catch (e) {
    v = await downloadFallbackData(url);
  }
  try {
    return await processDataInRes(v); //不再直接返回一个Promise 而是return await "promise" 注意产生的不同
  } catch (e) {
    return null;
  }
}
```

<table><td bgcolor=#F56C6C  >
return processDataInRes(v):出错时返回值为Promise而不是返回null。<br>

return await processDataInRes(v):出错时返回值为 null<br>

差异:return foo;不管 foo 是 promise 还是 rejects 都将会直接返回 foo。<br>
return await foo;如果 foo 是一个 Promise 将等待 foo 执行(resolve)或拒绝(reject)，如果是拒绝，将会在返回前抛出异常</td></table>
