---
title: Promise 并发限制
date: 2022/11/14
updated: 2021/11/14
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/弥豆子2.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/弥豆子2.jpg
description: Promise 并发池
---

# Promise 并发限制

## 背景

如果我们需要保证代码在多个异步事件后执行，会用到

```javascript
Promise.all(iterable);
```

`Promise.all` 可以保证，当 `interable` 参数（通常为 promises 数组）都达到 resolve 状态，则执行 `then` 回调

而 Promise 并发控制是指在每个时刻执行的 promise 数量是固定的（或者说小于 limit 值）

然而我们知道， `promise`的构造函数是  **<u>*同步执行*</u>**  的，也就是说传入到 `Promise.all`的多个 promise 实例，**在其创建的时候已经开始执行了**！

所以控制 promise 并发的关键，是控制 promise 的实例化

## 实现

上面提到，要实现 promise 并发控制，关键是控制 promise 实例

换句话说，就是把生成 promises 数组的控制权，交给并发控制逻辑

我们可以通过一个参数，接受 `并发任务数组`、`并发函数`、`并发数`三个参数，根据并发数监控 `promise` 的完成状态，批量创建新的 `promise`，从而达到控制 `promises` 生成的目的

## 代码实现

```javascript
/**
 *
 * @param {*} limit 并发限制数
 * @param {*} arr 并发任务数组
 * @param {*} fn 异步任务的构造函数
 */
const asyncPool = async (limit, arr, fn) => {
	const res = [] // 存储所有的 promise 实例
	const executing = [] // 存储正在执行的 promise 实例

	for (let item of arr) {
		// 调用构造函数 fn 实例化 promises，注意这里实例过程放在微任务里面
		const p = Promise.resolve().then(() => {
			fn(item, arr)
		})

		// 将异步任务 p 存入 res，由于 p 的实例化在微任务里，所以这里存储的 p 状态是 pending
		res.push(p)

		// 当并发任务数大于 limit 时，进行并发控制
		if (arr.length >= limit) {
			// 监听 promise 状态，达到完成状态后移除当前任务
			const e = p.finally(() => {
				executing.splice(executing.indexOf(e), 1)
			})
      // 将添加监听器的异步任务存入 executing，注意此时 e 状态为pending
			executing.push(e)

      /**
       * 当 executing.length 大于 limit 时，调用 Promise.race() 执行
       * Promise.race 的作用: 假如 poolLimit 是 2, executing 的任务有任意一个被解决,
       * Promise.race 就是 fulfilled 状态, 之后进入第 3 次 for 循环
       */
			if (executing.length >= limit) {
				await Promise.race(executing)
			}
		}
	}

  // 最后按序返回结果
	return Promise.all(res)
}
```



大概逻辑可以总结为

1. 先初始化 `limit` 个 promise 实例，将它们放到 `executing` 数组中
2. 使用 `Promise.race` 等待这 `limit` 个 promise 实例的执行结果
3. 一旦某一个 promise 的状态发生变更，就将其从 `executing` 中删除，然后再执行循环生成新的 promise，放入`executing` 中
4. 直到所有的 promise 都被执行完，最后使用 `Promise.all` 返回所有 promise 实例的执行结果



使用方式

```javascript
const timeout = (i) => new Promise((resolve) => setTimeout(() => resolve(i), i))

async function fn() {
	await asyncPool(2, [1000, 1000, 9000, 1000, 2000], timeout)
}
fn()
```

## 总结

所谓 promise 并发限制，实际上就是控制 promise 的实例化，如果是通过第三方函数，就把创建 promise 的控制权交给第三方即可
