---
title: methods，computed与watch的区别
date: 2021/9/7
updated: 2021/9/7
tags: Vue
categories: Vue
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/香奈乎1.png
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/香奈乎1.png
description: ' '
---

# methods，computed与watch的区别

**methods：**

- - 需要手动调用
  - 函数，所以是函数调用

**computed：**

- - 计算属性，所以是属性调用
  - 支持缓存
  - 只有依赖型数据发生改变，才会重新进行计算（数据必须是响应式的）
  - 不支持异步，当computed内有异步操作时无效，无法监听数据的变化
  - 应用场景：一个数据受多个数据影响

**watch：**

- - 是一个监听器，用于观察和响应数据变动
  - 不支持缓存
  - 只有依赖数据发生改变，才会重新进行计算（数据必须是响应式的）
  - 支持异步，
  - 应用场景：一个数据影响多个数据