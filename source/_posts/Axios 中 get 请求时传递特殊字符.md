---
title: Axios 中 get 请求时传递特殊字符
date: 2021/12/18
updated: 2021/12/18
tags: 
- 浏览器
- Axios
categories: Axios
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/magako.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/magako.jpg
---

# Axios 中 get 请求时传递特殊字符

## 起因

这周做一个筛选查询的需求，需要传递一个包含筛选条件的数组作为 `get` 请求的参数。由于后端设置的参数格式是对整个筛选条件进行编码。

举例：传递参数为

`where: [{"field":"dataType","op":"equal","value":"0"}]`时候(where为字段名)，

后端要求的格式是

`where=%5B%7B%22field%22%3A%22dataType%22%2C%22op%22%3A%22equal%22%2C%22value%22%3A%220%22%7D%5D`

## 初步方案

经过了解，对 URL 进行编码的有两种方法：`encodeURI()` 和 `encodeURIComponent()`

两者最大的区别就是 `encodeURI()`用于对一个完整的URL进行编码，而 `encodeURIComponent()`适用于对URL的组成部分进行个别编码，而不用与对整个URL进行编码

## 踩坑

于是，我自然而然地使用 `encodeURIComponent()` 对筛选条件进行编码。

然后，接口依然报错。打开控制台发现，中括号并没有被编码。

![](/img/6.png)

如果所示，不仅中括号没有被编码，连逗号也没有。

这就很难受，查看代码中的请求写法。

![](/img/1.png)

然而代码有条不紊地执行着，应该是OK的啊？

## 再探

无奈之下，打开 `Axios` 翻起来源码，找到了在创建 xhr 请求时，对请求参数做了一个 `buildUrl` 的处理。源码路径为： `axios/lib/helper/builderUrl.js` 。这个文件中我们可以找到对参数进行编码的函数

![](/img/2.png)

具体行为是将参数做了`encodeURIComponent`编码处理，但是，但是！！！然后又把下面的特殊字符转义回去了。所以我们可以看到逗号和中括号在请求的 URL 中没有被编码

## 最后

`Axios`做这样的处理也是情理之中，因为这些字符通常有着特殊的含义，所以要对其进行特殊转义处理。

那么我们怎么解决bug呢？

修改源码？自然是不可取的，太粗暴了。

**其实方法很简单，参数不放在 `params` 里面，就不会被 `Axios` 进行 特殊的编码处理。**

`Axios` 对 URL 的处理没有那么复杂，仅仅是执行了`combineURLs`,文件路径：(axios/lib/helpers/combineURLs.js)

![](/img/3.png)

**我们直接手动对参数进行编码然后拼接成字符串，再拼回 URL 中就可以完美解决这个问题了。**

![](/img/5.png)

再放一个修改前的代码，作为对比参考：

![](/img/4.png)

## 参考文章

[axios中get传参传递特殊字符](https://juejin.cn/post/6904504237422542862)

[axios get请求特殊字符编码问题](https://blog.csdn.net/liubangbo/article/details/112995555)