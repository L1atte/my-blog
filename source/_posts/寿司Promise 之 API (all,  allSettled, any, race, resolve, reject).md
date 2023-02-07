---
title: 寿司🍣 Promise 之 API (all,  allSettled, any, race, resolve, reject)
date: 2021/12/5
updated: 2021/12/5
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/碳治郎1.png
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/碳治郎1.png
description: ' '
---

# 寿司🍣 Promise 之 API (all,  allSettled, any, race, resolve, reject)


## 分析异同点

首先，我们对这些 API 进行分析，提取他们的共同之处，和发现他们的不同之处。

## 共同点

1. 参数都接受一个 promise 类型的`iterable`类型的输入（注：Array，Map，Set都属于ES6的`inerable`类型）
2. 这些方法都返回一个`Promise`实例

## 不同点

1. 返回的`Promise`实例的状态改变时机不同

   - `Promise.all`：在所有输入的 Promise 实例都`resolve`后执行自身的`resolve`回调，在任意一个输入的Promise实例`reject`后执行自身的`reject`回调
   - `Promise.allSettled`：在所有输入的 Promise 实例改变状态（`resolve`或者`reject`）后执行自身的`resolve`回调
   - `Promise.any`：在所有输入的 Promise 实例都`reject`后执行自身的`reject`回调，在任意一个输入的Promise实例`resolve`后执行自身的`resolve`回调
   - `Promise.race`：在任意一个输入的Promise实例改变状态后以相同的状态改变自身

2. 返回的Promise实例的终值（eventual value）或拒因（reason）不同

   - `Promise.all`方法返回的Promise实例终值是一个数组，数组的成员是所有输入的 Promise 实例的终值，并按照参数内的 promise 顺序排列，而不是由调用 promise 的完成顺序决定。拒因是输入的 Promise 实例中第一个状态变为`reject`的拒因

   - `Promise.allSettled`方法返回的 Promise 实例终值也是一个数组，并且按照参数内的`promise`顺序排列。其中的每个成员在输入promise为`resolved`状态时为`{status:'fulfilled', value:同一个终值}`，`rejected`状态时为`{status:'rejected', reason:同一个拒因}`

     ```javascript
     const promise1 = Promise.resolve(3);
     const promise2 = new Promise((resolve, reject) => setTimeout(reject, 100, 'foo'));
     const promises = [promise1, promise2];
     
     Promise.allSettled(promises).
       then((results) => results.forEach((result) => console.log(result)));
     // Object { status: "fulfilled", value: 3 }
     // Object { status: "rejected", reason: "foo" }
     ```

3. 参数为空迭代对象时，返回值不同

   - `Promise.all`：**同步**地返回一个已完成（resolved）状态的`promise`，其终值为空数组。（而且`Promise.all` **当且仅当**传入的可迭代对象为空时是**同步**）
   - `Promise.allSettled`：与`Promise.all`表现相同
   - `Promise.any`：**同步**地返回一个已失败（rejected）状态的 promise，其拒因是一个 `AggregateError` 对象。
   - `Promise.race`：返回一个永远等待的promise

## 用同一个思路处理这些 API

根据上文的异同点分析，我们需要对参数进行判断是否为`iterable`对象。定义一个结果收集数组和一个表示符合条件的 promise 状态个数变量，并且返回一个`Promise`实例。

### 通用模板：

```javascript
function template(promises) {
  if (promises.length === 0) {
    //  根据不同情况作处理
  }
  let result = [],
    num = 0;
  return new Promise((resolve, reject) => {
    const check = () => {
      if (num === promises.length) {
        //    根据不同情况调用 resolve 或 reject
      }
    };
    promises.forEach((item) => {
      Promise.resolve(item).then(
        (res) => {
          //  根据不同情况处理 result、num 和调用 resolve、reject、check 方法
        },
        (err) => {
          //  根据不同情况处理 result、num 和调用 resolve、reject、check 方法
        }
      );
    });
  });
}
```

### 插播一下 Promise.resolve 这个函数：

> Promise.resolve(value)方法返回一个以给定值解析后的 Promise 对象。如果这个值是一个 promise ，那么将返回这个 promise ；如果这个值是 thenable（即带有"then" 方法），返回的 promise 会“跟随”这个 thenable 的对象，采用它的最终状态；否则返回的 promise 将以此值完成。

因为 promises 的成员里可能混入了一些不是 promise 的值，所以用 Promise.resolve 去解析后就能统一为其添加 then 回调了。

## Promise.all

Promise.all：

- 传入的所有 Promsie 都是 fulfilled，则返回由他们的值组成的，状态为 fulfilled 的新 Promise
- 只要有一个 Promise 是 rejected，则返回 rejected 状态的新 Promsie，且它的值是第一个 rejected 的 Promise 的值
- 只要有一个 Promise 是 pending，则返回一个 pending 状态的新 Promise

```javascript
function all(promises) {
  if (promises.length === 0) {
    return Promise.resolve([]);
  }

  let result = [],
    num = 0;

  return new Promise((resolve, reject) => {
    const check = () => {
      if (num === promises.length) {
        resolve(result);
      }
    };

    promises.forEach((item, index) => {
      Promise.resolve(item).then(
        (res) => {
          num++;
          result[index] = res;
          check();
        },
        (err) => {
          reject(err);
        }
      );
    });
  });
}
```

## Promise.allSettled

Promise.allSettled：

- 所有 Promise 的状态都变化了，那么新返回一个状态是 fulfilled 的 Promise，且它的值是一个数组，数组的每项由所有 Promise 的值和状态组成的对象
- 如果有一个是 pending 的 Promise，则返回一个状态是 pending 的新实例

```javascript
function allSettled(promises) {
  if (promises.length === 0) {
    return Promise.resolve([]);
  }

  let result = [],
    sum = 0;

  return new Promise((resolve, reject) => {
    const check = () => {
      if (num === promises.length) {
        resolve(result);
      }
    };

    promises.forEach((item, index) => {
      Promise.resolve(item).then(
        (res) => {
          result[index] = { status: "fulfilled", value: res };
          num++;
          check();
        },
        (err) => {
          result[index] = { status: "rejected", reason: err };
          num++;
          check();
        }
      );
    });
  });
}
```

## Promise.any

Promise.any：

- 空数组或者所有 Promise 都是 rejected，则返回状态是 rejected 的新 Promsie，且值为 AggregateError 的错误
- 只要有一个是 fulfilled 状态的，则返回第一个是 fulfilled 的新实例
- 其他情况都会返回一个 pending 的新实例

```javascript
function any(promises) {
  if (promises.length === 0) {
    return Promise.reject(
      new AggregateError("No Promise in Promise.any was resolved")
    );
  }

  let result = [],
    num = 0;

  return new Promise((resolve, reject) => {
    const check = () => {
      if (num === result.length) {
        reject(new AggregateError("No Promise in Promise.any was resolved"));
      }
    };

    promises.forEach((item, index) => {
      Promise.resolve(item).then(
        (res) => {
          resolve(res);
        },
        (err) => {
          result[index] = err;
          num++;
          check();
        }
      );
    });
  });
}
```

## Promise.race

Promise.race：返回一个 promise，一旦迭代器中的某个promise解决或拒绝，返回的 promise就会解决或拒绝。

```javascript
function race(promises) {
  if (!promises.length) {
    throw Error("Promise.race need length");
  }

  return new Promise((resolve, reject) => {
    promises.forEach((item) => {
      Promise.resolve(item).then(
        (res) => {
          resolve(res);
        },
        (err) => {
          reject(err);
        }
      );
    });
  });
}
```

## Promise.resolve 和 Promise.reject

[Promsie.resolve(value) 可以将任何值转成值为 value ，状态是 fulfilled 的 Promise，但如果传入的值本身是 Promise 则会原样返回它。](https://bubuzou.com/2020/10/22/promise/)

```javascript
// Promise.resolve
Promise.resolve = function(value) {
  // 传入的值是 Promise
  if(value instanceof Promise) {
    return value
  }

  return new Promise(resolve => resolve(value))
}

// 如果传入的值本身是Promise则会原样返回他
let p = new Promise((resolve) => resolve(3));
console.log(p === Promise.resolve(p)); // true
```

[Promise.reject() 会实例化一个 rejected 状态的 Promise。但与 Promise.resolve() 不同的是，如果给 Promise.reject() 传递一个 Promise 对象，则这个对象会成为新 Promise 的值。](https://bubuzou.com/2020/10/22/promise/)

```javascript
// Promise.reject
Promise.reject = function(reason) {
  return new Promise((resolve, reject) =>{
    reject(reason)
  })
}

// 与 Promise.resolve() 不同的是，如果给 Promise.reject() 传递一个 Promise 对象，则这个对象会成为新 Promise 的值
let p = new Promise((resolve,reject) => reject(3));
console.log(p === Promise.reject(p)); // false
```

## 参考资料

[同一个套路手撕 Promise 的 all、allSettled、any、race 方法](https://juejin.cn/post/6948277034304929799)

[Promise.all()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

[Promise.allSettled()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

[Promise.any()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)

[Promise.race](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

[Promise.resolve()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)

[死磕 36 个 JS 手写题（搞懂后，提升真的大）](https://juejin.cn/post/6946022649768181774)

[深入理解Promise](https://bubuzou.com/2020/10/22/promise/)
