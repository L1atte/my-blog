---
title: 关于 React re-render 的心智模型和常见的认知误区
date: 2023/5/8
updated: 2023/5/8
tags: React
categories: React
description: ' '
---
# 关于 React re-render 的心智模型和常见的认知误区

## TL; DR

当且仅当组件（或其祖先之一）的**状态发生改变**的时候触发 re-render

> There are two reasons for a component to render:
>
> 1. It’s the component’s **initial render.**
> 2. The component’s (or one of its ancestors’) **state has been updated.**
>
> https://react.dev/learn/render-and-commit

## React 组件什么时候会触发 re-render？

组件 re-render 的原因可以归结为两个：

1. **自身状态改变**
2. 祖先**组件状态改变**

这其中还有一个很大的误区：一个组件会因为它的 props 改变而重新渲染。在后面我们会给出 case 来证明这点是不对的

## 重新渲染原因：状态改变

当一个组件的状态改变的时候，它就会进行重渲染。

[可以参阅 codesandbox 的例子](https://codesandbox.io/s/re-renders-because-of-state-38szcs?file=%2Fsrc%2FApp.tsx)

## 重新渲染原因：祖先组件状态改变

如果祖先组件发生重渲染，那么它将会导致其所有子组件重渲染

[可以参阅 codesandbox 的例子](https://codesandbox.io/s/re-renders-because-of-parent-21l638?file=%2Fsrc%2FApp.tsx)

[引用自官方 beta 文档](https://react.docschina.org/learn/render-and-commit)

> 在您触发渲染后，React 会调用您的组件来确定要在屏幕上显示的内容。**“渲染中” 即 React 在调用您的组件。**
>
> - **在进行初次渲染时,** React 会调用根组件。
> - **对于后续的渲染,** React 会调用内部状态更新触发了渲染的函数组件。
>
> 这个过程是递归的：如果更新后的组件会返回某个另外的组件，那么 React 接下来就会渲染 _那个_ 组件，而如果那个组件又返回了某个组件，那么 React 接下来就会渲染 _那个_ 组件，以此类推。这个过程会持续下去，直到没有更多的嵌套组件并且 React 确切知道哪些东西应该显示到屏幕上为止。

## 第一个误区：每当状态改变的时候，整个 React 树都会更新

有一部分人会认为，每当状态改变，整棵 fiber 树都会进行更新。

然而，官方文档提到：**对于后续的渲染,** React 会调用内部状态**更新触发了渲染的函数组件**。

[我们可以参考 codesandbox 的例子](https://codesandbox.io/s/not-re-render-all-the-tree-mvsz3z?file=%2Fsrc%2FApp.tsx)

在这个例子中，click 触发 `Count` 组件状态改变而导致 `Count` 自身和 `Child` 重渲染，并不会导致父组件 `App` 重渲染。

为什么父组件 `App` 没有重渲染呢？因为 React 的主要任务就是保持 React 内的状态和 React 渲染的 UI 的同步。React 更新，就是找出如何改变 UI，使其和新的状态同步。而在 React 中，数据是自上而下单向传递的（单向数据流，The Data Flows Down）。在这个例子中，`Count` 组件的状态 count 向下流向了 `Child` 组件的 prop `count`，但是不可能向上流向了 `App` 组件。因此，count 状态改变， `App` 组件并不需要更新。

## 第二个误区：组件会因为它的 props 改变而重新渲染

在讨论组件 re-render 的时候，props 是否改变并不重要。

因为实质上，为了改变 props，需要父组件状态改变。这将意味着父组件不得不重渲染，所以无论如何，子组件都会被强制重渲染，与 props 无关（不包括 `React.memo` ）

[请参考 codesandbox 的例子](https://codesandbox.io/s/re-render-props-not-relevant-z4ii6x?file=%2Fsrc%2FApp.tsx)

在例子中，`Child` 组件的 props.value 是一个全局常量——这意味着 props 不会改变

然而，在祖先组件 App 状态改变的时候，`Child` 组件**仍然进行了重渲染**！

仅当使用 memoization techniques （`React.memo` , `useMemo` ）的时候，props 的改变才显得重要
