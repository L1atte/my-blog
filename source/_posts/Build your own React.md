---
title: Build your own React
date: 2023/2/15
updated: 2023/2/15
tags: React
categories: React
description: "从零实现简易的 React"
---

递归调用 render 渲染节点的缺点：一旦开始 render，直到结束前我们都没有方法中断渲染。当节点树很大的时候，我们可能需要等待很长时间

所以我们将整个工作分成一个个小的工作单元，当我们完成小工作单元并且没有其他要做的事情时，我们会让出线程给浏览器渲染。因此，我们使用一个新的数据结构 fiber，一个 fiber 意味着一个 dom 节点，也意味着一个小的工作单元

我们在 render 方法里面定义 rootFiber，并将其设置为 `nextUnitOfWork`，后续在`performUnitOfWork`方法中获取工作单元

每个 fiber 会做三件事

1. 将元素添加进 dom
2. 为当前 fiber 的 children 创建 fiber
3. 选择下一个工作单元

fiber 结构的好处是很轻松地找到下一个工作单元（具有 parent、children、sibling 节点引用）

当我们完成一个 fiber 的工作时，我们会按照以下优先级选择下一个 fiber

- 在当前节点下寻找是否有子节点
  - 若有, 则进入子节点
  - 若没有, 则在当前节点下寻找是否有下一个相邻节点
    - 若有, 则进入下一个相邻节点
    - 若没有, 则返回它的父节点

当我们一直遍历到 rootFiber，意味着 `render`方法结束了

```jsx
nextUnitOfWork = {
  dom: container,
  props: {
    children: [element],
  },
};

const fiber = {
  dom,
  props: {
    children,
  },
};

if (!fiber.dom) {
  fiber.dom = createDom(fiber);
}

if (fiber.parent) {
  fiber.parent.dom.appendChild(fiber.dom);
}
```

# 从零实现简易的 React

## The `createElement`Function

通常，我们在 React 中使用 JSX 定义组件。这实际上会被 babel 转译成调用 `React.createElement( )`的形式

```jsx
const element = <h1 title="foo">Hello</h1>;
```

`React.createElement( )`会从其参数创建一个对象，所以我们可以直接写成

```jsx
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
// React.createElement( )返回的结果：
{
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
```

`React.createElement( )`返回一个具有`type`和`props`字段的对象

`type`是一个字符串，制定我们要创建的 DOM 节点的类型，被用于传递给 `document.createElement`创建 HTML 元素，它也可以是一个函数，我们会在 xxx 提到

`props`是一个对象，它有所有来自 `JSX`属性的键和值。同时，它还有一个特殊的属性：`children`

我们手动实现一下`React.createElement`，通过对`props`使用展开操作符、对`children`使用剩余参数，使得`children`始终为数组,当`children`包含基础类型的时候，我们为其指定一个特殊类型`TEXT_ELEMENT`

```jsx
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child => {
        return typeof child === "object" ? child : createTextElement(child);
      }),
    },
  };
}
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}
```

例如，`createElement('div')`会 return

```jsx
{
  "type": "div",
  "props": { "children": [] }
}
```

`createElement("div", null, a, b)`会 return

```jsx
{
  "type": "div",
  "props": { "children": [a, b] }
}
```

> React 没有包裹原始值，也没有在没有孩子的时候创建空数组，但我们这样做是因为它会简化我们的代码，对于我们的库来说，我们更喜欢简单的代码，而不是性能好的代码。

## The `render` Function

我们来实现下`React.render()`，首先，我们只关心如何向`DOM`添加东西，后续再处理更新和删除

```jsx
function render(element, container) {
  // 根据 type 创建 dom
  const dom = element.type == "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(element.type);

  // 将 props 分配给 dom
  const isProperty = key => key !== "children";
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name];
    });

  // 递归处理 children
  element.props.children.forEach(child => render(child, dom));
  // 将 dom 插入 container
  container.appendChild(dom);
}
```

首先，我们根据元素的`type`创建一个`dom`，然后将`props`分配给`dom`，最后递归处理元素的`children`，

## Concurrent Mode

但是，我们需要重构一下之前的代码

问题出在递归调用`render`上

一旦我们开始渲染，将无法阻止渲染一`dom`树。而且，如果这棵树很大，将会长期阻塞主线程。如果浏览器需要做一些高优先级的事情，比如处理用户的输入或保持动画的流畅，它就必须等到渲染完成。

因此，我们将整个`render`任务分成一个个小单元，在完成每个单元后，如果还有别的需要做，我们会让浏览器打断渲染

我们使用`requestIdleCallback`来完成这一调度

```jsx
let nextUnitOfWork = null;

function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }
  requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

> 关于`requestIdleCallback`，这里参考[你应该知道的 requestIdleCallback](https://juejin.im/post/5ad71f39f265da239f07e862)。

> React 不再使用 `requestIdleCallback`了，原因见：https://github.com/facebook/react/issues/11171#issuecomment-417349573。
>
> 现在它使用 scheduler：https://github.com/facebook/react/tree/main/packages/scheduler。但对于这个用例来说，它在概念上是一样的。

## Fibers

为了组织每个工作单元，我们定义一个数据结构：fiber tree

每个`fiber`对应一个`element`，而且`fiber`也是一个工作单元

我们会在`render`方法里面设置`root fiber`，并且将其设置为`nextUnitOfWork`。而其余的工作将在`performUnitOfWork`函数里发生，在那里我们为每一个`fiber`做三件事

1. 将`elemenr`添加进`dom`
2. 在`element.children`里创建`fiber`
3. 选择下一个工作单元（这也是`fiber`结构的目的之一）

现在我们用代码来实现

首先，需要调整`render`的代码，将创建`dom`的逻辑抽离，因为上面提到了我们将创建`dom`的工作交给 fiber 完成

```jsx
function createDom(fiber) {
  const dom = fiber.type == "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(fiber.type);

  const isProperty = key => key !== "children";
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name];
    });

  return dom;
}
```

然后，我们创建一个新的全局变量`nextUnitOfWork`，并在`render`函数中将其设置为`root fiber`

```jsx
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  };
}
let nextUnitOfWork = null;
```

然后，当浏览器就绪时，会通过`requestIdleCallback`调用`workLoop`函数，我们将开始在`root fiber`工作

```jsx
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }
  requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
  // TODO add dom node
  // TODO create new fibers
  // TODO return next unit of work
}
```

首先，我们创建一个新的节点并将其追加到`dom`中，并在`fiber.dom`中保持对节点的追踪

```jsx
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber);
  }
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom);
  }

  // TODO create new fibers
  // TODO return next unit of work
}
```

接着我们为 children 创建 newFiber ：如果 newFiber 是第一个孩子，那么他是 child，其余都是 sibling

```jsx
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber);
  }
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom);
  }

  const elements = fiber.props.children;
  let prevSibling = null;
  for (let i = 0; i < elements.length - 1; i++) {
    const element = elements[i];
    const newFiber = {
      type: element.type,
      props: elements.props,
      parent: fiber,
      dom: null,
    };

    if (i === 0) {
      fiber.child = newFiber;
    } else {
      fiber.sibling = newFiber;
    }
    prevSibling = newFiber;
  }
  // TODO return next unit of work
}
```

最后我们选择下一个工作单元，我们会按照以下优先级选择下一个 fiber

- 在当前节点下寻找是否有子节点
  - 若有, 则进入子节点
  - 若没有, 则在当前节点下寻找是否有下一个相邻节点
    - 若有, 则进入下一个相邻节点
    - 若没有, 则返回它的父节点

```jsx
function performUnitOfWork(fiber) {
  // 省略前面的代码

  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.parent;
  }
}
```

## Render and Commit Phases

```jsx
function performUnitOf(fiber) {
  // 省略前面的代码

  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom);
  }
}
```

如上所示，我们在对每个元素进行处理时，都会向`dom`添加一个新的节点

而且，由于使用`requestIdleCallback`进行调度的原因，在我们完成整棵树的渲染工作前，浏览器可能会中断我们的工作。

在这种情况下，用户会看到一个不完整的界面，这是我们不希望的

所以我们从将提交`DOM`的逻辑抽离，在完成整个树的渲染工作后再提交`DOM`（称为 Commit phases）

并且，我们将会追踪`fiber tree`的`root fiber`，称之为 work in progress root -- `wipRoot`

```jsx
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  };
  nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let wipRoot = null;
```

一旦我们完成所有工作，我们就将`fiber tree`提交给`DOM`

```jsx
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }

  if (!nextUnitOfWork && wipRoot) {
    // 提交阶段
    commitRoot();
  }

  requestIdleCallback(workLoop);
}
```

在`commitRoot`函数中，我们递归地向`DOM`中插入节点

```jsx
function commitRoot() {
  commitWork(wipRoot);
  wipRoot = null;
}
function commitWork(fiber) {
  if (!fiber) {
    return;
  }
  const domParent = fiber.parent.dom;
  domParent.appendChild(fiber.dom);
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```
