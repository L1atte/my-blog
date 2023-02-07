---
title: 寿司🍣Promise/A+，并通过promise-test测试
date: 2021/12/5
updated: 2021/12/5
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/心海.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/心海.jpg
description: ' '
---

# 寿司🍣Promise/A+，并通过promise-test测试


## Promise/A+ 规范

首先，我们需要明确一下，手撕`Promise`，到底要写到一个什么程度？

此处我们可以参考一个 commonjs 社区提出的 [Promise/A+](https://promisesaplus.com/) 规范，根据规范，我们可以一步步从易到难写出一个简单的 Promise 实现。

## 术语

- `Promise`：是一个拥有 `then` 方法的对象或者函数
- `value`：指任何 JavaScript 的合法值（包括 `undefined`， "thenable" 和 "promise"）
- `reason`：是一个值，表示承诺被拒绝的原因

## 几个实现的关键点

- Promise的状态只能是下面三者之一：pending(待定)，fulfilled(解决)，rejected(拒绝)。
- Then方法的接收两个参数，接受 onFulfilled 和 onRejected 参数。
- onFulfilled 和 onRejected 需要被异步调用，这里用 setTimeout 模拟异步
- 执行函数`executor`为异步函数的时候，用 `onResolvedCallbacks` 和 `onRejectedCallbacks` 两个数组分别把成功和失败的回调存起来
- then 需要支持链式调用，所以得返回一个新的 Promise
- 为了让链式调用正常进行下去，需要判断 onFulfilled 和 onRejected 的类型

## 具体代码

```javascript
// promise.js
class Promise {
  static PENDING = "pending";
  static RESOLVED = "resolved";
  static REJECTED = "rejected";

  constructor(executor) {
    this.state = Promise.PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];

    let resolve = (value) => {
      if (this.state === Promise.PENDING) {
        this.state = Promise.RESOLVED;
        this.value = value;
        this.onResolvedCallbacks.forEach((callback) => callback());
      }
    };

    let reject = (reason) => {
      if (this.state === Promise.PENDING) {
        this.state = Promise.REJECTED;
        this.reason = reason;
        this.onRejectedCallbacks.forEach((callback) => callback());
      }
    };

    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }

  then(onFulfilled, onRejected) {
    // 解决 onFulfilled，onRejected 没有传值的问题
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
    // 因为错误的值要让后面访问到，所以这里也要抛出错误，不然会在之后 then 的 resolve 中捕获
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (err) => {
            throw err;
          };

    // 每次调用 then 都返回一个新的 promise
    let promise2 = new Promise((resolve, reject) => {
      if (this.state === Promise.RESOLVED) {
        //Promise/A+ 2.2.4 --- setTimeout
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            // x可能是一个promise
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        }, 0);
      }

      if (this.state === Promise.REJECTED) {
        //Promise/A+ 2.2.3
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        }, 0);
      }

      if (this.state === Promise.PENDING) {
        this.onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });

        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });
      }
    });

    return promise2;
  }
}

const resolvePromise = (promise2, x, resolve, reject) => {
  // 自己等待自己完成是错误的实现，用一个类型错误，结束掉 promise  Promise/A+ 2.3.1
  if (promise2 === x) {
    return reject(
      new TypeError("Chaining cycle detected for promise #<Promise>")
    );
  }
  // Promise/A+ 2.3.3.3.3 只能调用一次  
  let called = false;
  if ((typeof x === "object" && x !== null) || typeof x === "function") {
    try {
      // Promise/A+ 2.3.3.1
      let then = x.then;
      if (typeof then === "function") {
        // Promise/A+ 2.3.3.3
        // 不要写成 x.then，直接 then.call 就可以了 因为 x.then 会再次取值，Object.defineProperty
        then.call(
          x,
          (y) => {
            if (called) return;
            called = true;
            // Promise/A+ 2.3.3.3.1
            // 递归解析的过程（因为可能 promise 中还有 promise）
            resolvePromise(promise2, y, resolve, reject);
          },
          (r) => {
            // 只要失败就失败 Promise/A+ 2.3.3.3.2
            if (called) return;
            called = true;
            reject(r);
          }
        );
      } else {
        // Promise/A+ 2.3.3.4
        // 如果 then 不是函数，以 x 为参数解决 promise2
        resolve(x);
      }
    } catch (error) {
      // Promise/A+ 2.3.3.2
      if (called) return;
      called = true;
      reject(error);
    }
  } else {
    // Promise/A+ 2.3.4
    // 如果 x 不为对象或者函数，以 x 为参数解决 promise2
    resolve(x);
  }
};

module.exports = Promise;
```

## 添加测试用例

Promise写完后可以通过 promises-aplus-tests 这个包进行测试，看是不是符合 Promise/A+ 规范。

在测试前，我们添加一段代码

```javascript
// promise.js
// 这里是上面写的 Promise 全部代码
Promise.defer = Promise.deferred = function () {
    let dfd = {}
    dfd.promise = new Promise((resolve,reject)=>{
        dfd.resolve = resolve;
        dfd.reject = reject;
    });
    return dfd;
}
module.exports = Promise;
```

安装 promises-aplus-tests

```shell
npm init
npm i promises-aplus-tests

// 安装后修改 package.json 的 scripts 处的代码，添加"test": "promises-aplus-tests promise.js"
// 在终端运行
npm test
```

代码无误后会显示  `872 passing`