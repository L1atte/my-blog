---
title: å¯¿å¸ð£ Promise ä¹ API (all,  allSettled, any, race, resolve, reject)
date: 2021/12/5
updated: 2021/12/5
tags: JavaScript
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/ç¢³æ²»é1.png
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/ç¢³æ²»é1.png
---

# å¯¿å¸ð£ Promise ä¹ API (all,  allSettled, any, race, resolve, reject)


## åæå¼åç¹

é¦åï¼æä»¬å¯¹è¿äº API è¿è¡åæï¼æåä»ä»¬çå±åä¹å¤ï¼ååç°ä»ä»¬çä¸åä¹å¤ã

## å±åç¹

1. åæ°é½æ¥åä¸ä¸ª promise ç±»åç`iterable`ç±»åçè¾å¥ï¼æ³¨ï¼Arrayï¼Mapï¼Seté½å±äºES6ç`inerable`ç±»åï¼
2. è¿äºæ¹æ³é½è¿åä¸ä¸ª`Promise`å®ä¾

## ä¸åç¹

1. è¿åç`Promise`å®ä¾çç¶ææ¹åæ¶æºä¸å

   - `Promise.all`ï¼å¨ææè¾å¥ç Promise å®ä¾é½`resolve`åæ§è¡èªèº«ç`resolve`åè°ï¼å¨ä»»æä¸ä¸ªè¾å¥çPromiseå®ä¾`reject`åæ§è¡èªèº«ç`reject`åè°
   - `Promise.allSettled`ï¼å¨ææè¾å¥ç Promise å®ä¾æ¹åç¶æï¼`resolve`æè`reject`ï¼åæ§è¡èªèº«ç`resolve`åè°
   - `Promise.any`ï¼å¨ææè¾å¥ç Promise å®ä¾é½`reject`åæ§è¡èªèº«ç`reject`åè°ï¼å¨ä»»æä¸ä¸ªè¾å¥çPromiseå®ä¾`resolve`åæ§è¡èªèº«ç`resolve`åè°
   - `Promise.race`ï¼å¨ä»»æä¸ä¸ªè¾å¥çPromiseå®ä¾æ¹åç¶æåä»¥ç¸åçç¶ææ¹åèªèº«

2. è¿åçPromiseå®ä¾çç»å¼ï¼eventual valueï¼ææå ï¼reasonï¼ä¸å

   - `Promise.all`æ¹æ³è¿åçPromiseå®ä¾ç»å¼æ¯ä¸ä¸ªæ°ç»ï¼æ°ç»çæåæ¯ææè¾å¥ç Promise å®ä¾çç»å¼ï¼å¹¶æç§åæ°åç promise é¡ºåºæåï¼èä¸æ¯ç±è°ç¨ promise çå®æé¡ºåºå³å®ãæå æ¯è¾å¥ç Promise å®ä¾ä¸­ç¬¬ä¸ä¸ªç¶æåä¸º`reject`çæå 

   - `Promise.allSettled`æ¹æ³è¿åç Promise å®ä¾ç»å¼ä¹æ¯ä¸ä¸ªæ°ç»ï¼å¹¶ä¸æç§åæ°åç`promise`é¡ºåºæåãå¶ä¸­çæ¯ä¸ªæåå¨è¾å¥promiseä¸º`resolved`ç¶ææ¶ä¸º`{status:'fulfilled', value:åä¸ä¸ªç»å¼}`ï¼`rejected`ç¶ææ¶ä¸º`{status:'rejected', reason:åä¸ä¸ªæå }`

     ```javascript
     const promise1 = Promise.resolve(3);
     const promise2 = new Promise((resolve, reject) => setTimeout(reject, 100, 'foo'));
     const promises = [promise1, promise2];
     
     Promise.allSettled(promises).
       then((results) => results.forEach((result) => console.log(result)));
     // Object { status: "fulfilled", value: 3 }
     // Object { status: "rejected", reason: "foo" }
     ```

3. åæ°ä¸ºç©ºè¿­ä»£å¯¹è±¡æ¶ï¼è¿åå¼ä¸å

   - `Promise.all`ï¼**åæ­¥**å°è¿åä¸ä¸ªå·²å®æï¼resolvedï¼ç¶æç`promise`ï¼å¶ç»å¼ä¸ºç©ºæ°ç»ãï¼èä¸`Promise.all` **å½ä¸ä»å½**ä¼ å¥çå¯è¿­ä»£å¯¹è±¡ä¸ºç©ºæ¶æ¯**åæ­¥**ï¼
   - `Promise.allSettled`ï¼ä¸`Promise.all`è¡¨ç°ç¸å
   - `Promise.any`ï¼**åæ­¥**å°è¿åä¸ä¸ªå·²å¤±è´¥ï¼rejectedï¼ç¶æç promiseï¼å¶æå æ¯ä¸ä¸ª `AggregateError` å¯¹è±¡ã
   - `Promise.race`ï¼è¿åä¸ä¸ªæ°¸è¿ç­å¾çpromise

## ç¨åä¸ä¸ªæè·¯å¤çè¿äº API

æ ¹æ®ä¸æçå¼åç¹åæï¼æä»¬éè¦å¯¹åæ°è¿è¡å¤æ­æ¯å¦ä¸º`iterable`å¯¹è±¡ãå®ä¹ä¸ä¸ªç»ææ¶éæ°ç»åä¸ä¸ªè¡¨ç¤ºç¬¦åæ¡ä»¶ç promise ç¶æä¸ªæ°åéï¼å¹¶ä¸è¿åä¸ä¸ª`Promise`å®ä¾ã

### éç¨æ¨¡æ¿ï¼

```javascript
function template(promises) {
	if (promises.length === 0) {
		//  æ ¹æ®ä¸åæåµä½å¤ç
	}
	let result = [],
		num = 0;
	return new Promise((resolve, reject) => {
		const check = () => {
			if (num === promises.length) {
				//    æ ¹æ®ä¸åæåµè°ç¨ resolve æ reject
			}
		};
		promises.forEach((item) => {
			Promise.resolve(item).then(
				(res) => {
					//  æ ¹æ®ä¸åæåµå¤ç resultãnum åè°ç¨ resolveãrejectãcheck æ¹æ³
				},
				(err) => {
					//  æ ¹æ®ä¸åæåµå¤ç resultãnum åè°ç¨ resolveãrejectãcheck æ¹æ³
				}
			);
		});
	});
}
```

### ææ­ä¸ä¸ Promise.resolve è¿ä¸ªå½æ°ï¼

> Promise.resolve(value)æ¹æ³è¿åä¸ä¸ªä»¥ç»å®å¼è§£æåç Promise å¯¹è±¡ãå¦æè¿ä¸ªå¼æ¯ä¸ä¸ª promise ï¼é£ä¹å°è¿åè¿ä¸ª promise ï¼å¦æè¿ä¸ªå¼æ¯ thenableï¼å³å¸¦æ"then" æ¹æ³ï¼ï¼è¿åç promise ä¼âè·éâè¿ä¸ª thenable çå¯¹è±¡ï¼éç¨å®çæç»ç¶æï¼å¦åè¿åç promise å°ä»¥æ­¤å¼å®æã

å ä¸º promises çæåéå¯è½æ··å¥äºä¸äºä¸æ¯ promise çå¼ï¼æä»¥ç¨ Promise.resolve å»è§£æåå°±è½ç»ä¸ä¸ºå¶æ·»å  then åè°äºã

## Promise.all

Promise.allï¼

- ä¼ å¥çææ Promsie é½æ¯ fulfilledï¼åè¿åç±ä»ä»¬çå¼ç»æçï¼ç¶æä¸º fulfilled çæ° Promise
- åªè¦æä¸ä¸ª Promise æ¯ rejectedï¼åè¿å rejected ç¶æçæ° Promsieï¼ä¸å®çå¼æ¯ç¬¬ä¸ä¸ª rejected ç Promise çå¼
- åªè¦æä¸ä¸ª Promise æ¯ pendingï¼åè¿åä¸ä¸ª pending ç¶æçæ° Promise

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

Promise.allSettledï¼

- ææ Promise çç¶æé½ååäºï¼é£ä¹æ°è¿åä¸ä¸ªç¶ææ¯ fulfilled ç Promiseï¼ä¸å®çå¼æ¯ä¸ä¸ªæ°ç»ï¼æ°ç»çæ¯é¡¹ç±ææ Promise çå¼åç¶æç»æçå¯¹è±¡
- å¦ææä¸ä¸ªæ¯ pending ç Promiseï¼åè¿åä¸ä¸ªç¶ææ¯ pending çæ°å®ä¾

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

Promise.anyï¼

- ç©ºæ°ç»æèææ Promise é½æ¯ rejectedï¼åè¿åç¶ææ¯ rejected çæ° Promsieï¼ä¸å¼ä¸º AggregateError çéè¯¯
- åªè¦æä¸ä¸ªæ¯ fulfilled ç¶æçï¼åè¿åç¬¬ä¸ä¸ªæ¯ fulfilled çæ°å®ä¾
- å¶ä»æåµé½ä¼è¿åä¸ä¸ª pending çæ°å®ä¾

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

Promise.raceï¼è¿åä¸ä¸ª promiseï¼ä¸æ¦è¿­ä»£å¨ä¸­çæä¸ªpromiseè§£å³ææç»ï¼è¿åç promiseå°±ä¼è§£å³ææç»ã

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

## Promise.resolve å Promise.reject

[Promsie.resolve(value) å¯ä»¥å°ä»»ä½å¼è½¬æå¼ä¸º value ï¼ç¶ææ¯ fulfilled ç Promiseï¼ä½å¦æä¼ å¥çå¼æ¬èº«æ¯ Promise åä¼åæ ·è¿åå®ã](https://bubuzou.com/2020/10/22/promise/)

```javascript
// Promise.resolve
Promise.resolve = function(value) {
  // ä¼ å¥çå¼æ¯ Promise
  if(value instanceof Promise) {
    return value
  }

  return new Promise(resolve => resolve(value))
}

// å¦æä¼ å¥çå¼æ¬èº«æ¯Promiseåä¼åæ ·è¿åä»
let p = new Promise((resolve) => resolve(3));
console.log(p === Promise.resolve(p)); // true
```

[Promise.reject() ä¼å®ä¾åä¸ä¸ª rejected ç¶æç Promiseãä½ä¸ Promise.resolve() ä¸åçæ¯ï¼å¦æç» Promise.reject() ä¼ éä¸ä¸ª Promise å¯¹è±¡ï¼åè¿ä¸ªå¯¹è±¡ä¼æä¸ºæ° Promise çå¼ã](https://bubuzou.com/2020/10/22/promise/)

```javascript
// Promise.reject
Promise.reject = function(reason) {
  return new Promise((resolve, reject) =>{
    reject(reason)
  })
}

// ä¸ Promise.resolve() ä¸åçæ¯ï¼å¦æç» Promise.reject() ä¼ éä¸ä¸ª Promise å¯¹è±¡ï¼åè¿ä¸ªå¯¹è±¡ä¼æä¸ºæ° Promise çå¼
let p = new Promise((resolve,reject) => reject(3));
console.log(p === Promise.reject(p)); // false
```

## åèèµæ

[åä¸ä¸ªå¥è·¯ææ Promise ç allãallSettledãanyãrace æ¹æ³](https://juejin.cn/post/6948277034304929799)

[Promise.all()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

[Promise.allSettled()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

[Promise.any()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)

[Promise.race](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

[Promise.resolve()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)

[æ­»ç£ 36 ä¸ª JS æåé¢ï¼ææåï¼æåççå¤§ï¼](https://juejin.cn/post/6946022649768181774)

[æ·±å¥çè§£Promise](https://bubuzou.com/2020/10/22/promise/)
