---
title: Build your own React
date: 2023/2/15
updated: 2023/2/15
tags: React
categories: React
description: ' '
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
  domParent.appendChild(fiber.dom); // 我们只有向 DOM 添加节点，更新和删除呢？见下一节
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

## Reconciliation

目前为止，我们只是向 DOM 中添加节点，但是更新和删除节点呢？

这就是我们接下来要做的，我们需要将在`render`函数收到的`elements`与提交给`dom`的最后一个`fiber tree`做比较

我们添加一个全局变量`currentRoot`，保存在完成`commit`阶段后提交到`dom`的最后一个`fiber tree`，并且在每一个`fiber`中添加`alternate`属性，指向前一个`commit`阶段的`fiber`

```jsx
function commitRoot() {
	commitWork(wipRoot.child);
	currentRoot = wipRoot; // 完成 commit 后保存引用
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

function render(element, container) {
	wipRoot = {
		dom: container,
		props: {
			children: [element],
		},
		alternate: currentRoot, // 上次 commit 的 fiber
	};
	nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null; // 上次渲染的 fiber tree
let wipRoot = null;
```

然后我们对`performUnitOfWork`函数进行重构，添加一个`reconcileChildren`函数，对原先`create new fiber`的代码进行重构

```jsx
function performUnitOfWork(fiber) {
	if (!fiber.dom) {
		fiber.dom = createDom(fiber);
	}

	const elements = fiber.props.children;
	reconcileChildren(fiber, elements); // reconcileChildren

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

在`reconcileChildren`函数中对比旧的`fiber`和新的`elements`

我们同时迭代旧的`fiber`的子节点和新元素的子节点

我们先忽略同时迭代两者的逻辑，只关心最重要的东西：`oldFiber`和`element`。`element`是我们即将要渲染到 DOM 的东西，而`oldFiber`是上次渲染的内容

我们需要对它们进行比较，看看是否有任何需要应用于DOM的变化。

使用`type`来比较它们

- 如果类型相同，我们保持旧`fiber`的引用，然后仅更新`props`
- 如果类型不同、存在新`element`，意味着需要创建新的`dom`节点
- 如果类型不同、不存在新`element`、有旧`fiber`，意味着需要删除旧的`dom`节点

> 实际上，React 也是用了 key，这会让 diff 算法更准确。例如：它可以检测 children 的位置变化

```jsx
function reconcileChildren(wipFiber, elements) {
	oldFiber = wipFiber.alternate && wipFiber.alternate.child;
	let prevSibling = null;

	while (index < elements.length || oldFiber != null) {
		const element = elements[index];
		let newFiber = null;
		const sameType = oldFiber && element && element.type === oldFiber.type;

		if (sameType) {
			// TODO update the node
		}
		if (element && !sameType) {
			// TODO add this node
		}
		if (oldFiber && !sameType) {
			// TODO delete the oldFiber's node
		}

		// 遍历 fiber
		// 因为 fiber 只有一个 child 节点，其余 child 节点被视作 child 节点的 sibling 节点。
		// 因此这样遍历
		if (oldFiber) oldFiber = oldFiber.sibling;

		// 第一个child才可以作为child，其他的就是sibling
		if (index === 0) {
			wipFiber.child = newFiber;
		} else {
			prevSibling.sibling = newFiber;
		}

		prevSibling = newFiber;
		index++;
	}
}
```

当旧 fiber 和新元素有着相同的类型的时候，我们创建一个新 fiber，它保持旧 fiber 的 dom 节点

我们还为 fiber 添加了一个新属性：`effectTag`，我们将在后面的 commit phase 使用这个属性

```javascript
// update the node
if (sameType) {
	newFiber = {
		type: oldFiber.type,
		props: element.props,
		dom: oldFiber.dom,
		parent: wipFiber,
		alternate: oldFiber,
		effectTag: "UPDATE",
	};
}
```

对于元素需要一个新的 dom 节点的情况，我们使用 `PLACEMENT`的`effectTag`来标记他

```jsx
// add the node
if (!sameType && element) {
	newFiber = {
		type: element.type,
		props: element.props,
		dom: null,
		parent: wipFiber,
		alternate: null,
		effectTag: "PLACEMENT",
	};
}
```

对于需要删除节点的情况，由于我们没有新的元素，所以把`effectTag`加在旧 fiber 上

但是当我们将 fiber tree 提交到 dom 时，我们将会从 work in progress root (wipRoot) 开始，那里没有旧 fiber



```jsx
if (!sameType && oldFiber) {
	oldFiber.effectTag = "DELETION";
	deletions.push(oldFiber);
}
```

所以我们添加一个全局变量`deletions`来记录要删除的 fiber

```jsx
function render(element, container) {
	wipRoot = {
		dom: container,
		props: {
			children: [element],
		},
		alternate: currentRoot,
	};
	deletions = [];
	nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null;
let wipRoot = null;
let deletions = null; // 记录要删除的 fiber
```

然后，当我们向 dom 提交变化的时候，会使用 `deletions`

```jsx
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
```

现在，让我们在`commitWork`函数中添加处理`effectTag`的逻辑

```jsx
function commitWork(fiber) {
	if (!fiber) {
		return;
	}
	const domParent = fiber.parent.dom;
	if (fiber.effectTag === "PLACEMENT" && fiber.dom != null) {
		domParent.appendChild(fiber.dom);
	} else if (fiber.effectTag === "UPDATE" && fiber.dom != null) {
		updateDom(fiber.dom, fiber.alternate.props, fiber.props);
	} else if (fiber.effectTag === "DELETION") {
		domParent.removeChild(fiber.dom);
	}

	commitWork(fiber.child);
	commitWork(fiber.sibling);
}
function updateDom(dom, prevProps, nextProps) {
  // TODO
}
```

在`UPDATE`的时候，我们需要删除已经消失的属性、并且设置新增或者修改的属性

```jsx
function updateDom(dom, prevProps, nextProps) {
	const isEvent = key => key.startsWith("on");
	// 删除已经没有的props
	Object.keys(prevProps)
		.filter(key => key != "children" && !isEvent(key))
		// 不在nextProps中
		.filter(key => !key in nextProps)
		.forEach(key => {
			// 清空属性
			dom[key] = "";
		});

	// 添加新增的属性/修改变化的属性
	Object.keys(nextProps)
		.filter(key => key !== "children" && !isEvent(key))
		// 不再prevProps中
		.filter(key => !key in prevProps || prevProps[key] !== nextProps[key])
		.forEach(key => {
			dom[key] = nextProps[key];
		});

	// 删除事件处理函数
	Object.keys(prevProps)
		.filter(isEvent)
		// 新的属性没有，或者有变化
		.filter(key => !key in nextProps || prevProps[key] !== nextProps[key])
		.forEach(key => {
			const eventType = key.toLowerCase().substring(2);
			dom.removeEventListener(eventType, prevProps[key]);
		});

	// 添加新的事件处理函数
	Object.keys(nextProps)
		.filter(isEvent)
		.filter(key => prevProps[key] !== nextProps[key])
		.forEach(key => {
			const eventType = key.toLowerCase().substring(2);
			dom.addEventListener(eventType, nextProps[key]);
		});
}
```

## Function Components

接下来我们添加对函数式组件的支持

**函数式组件相当于包裹了函数的 jsx 元素**

```jsx
function App(props) {
  return <h1>Hi {props.name}</h1>
}
const element = <App name="foo" />
```

首先我们将这段 jsx 代码转换成 js 的形式

```jsx
function App(props) {
	return Didact.createElement("h1", null, "Hi ", props.name);
}
const element = Didact.createElement(App, {
	name: "foo",
});
```

Function components 有两点不同

1. 来自函数式组件的 fiber 没有 dom 节点（缺少和 dom 类型对应的 fiber.type 字段，因为 fiber.type 实际是一个函数）
2. `children`来自于运行中的函数而不是`props`

因此，我们在`performUnitOfWork`函数中检查 fiber 是否是一个函数，根据这一点，我们选择不同的更新函数

```jsx
function performUnitOfWork(fiber) {
  // 根据 fiber.type 决定不同的更新函数
	const isFunctionComponent = fiber.type instanceof Function;
	if (isFunctionComponent) updateFunctionComponent(fiber);
	else updateHostComponent(fiber);

	// ......rest code
}
```

在`updateHostCompenent`函数中我们的做法与之前一样

```jsx
function updateHostComponent(fiber) {
	// add dom node
	if (!fiber.dom) fiber.dom = createDom(fiber);

	// 新建 newFiber，构建 fiber tree
	reconcileChildren(fiber, fiber.props.children);
}
```

然后我们在`updateFunctionComponent`函数中获取`children`

```jsx
function updateFunctionComponent(fiber) {
	// 这里 fiber.type 是一个函数
	const children = [fiber.type(fiber.props)];

	// 新建 newFiber，构建 fiber tree
	reconcileChildren(fiber, children);
}
```

用之前的例子解释一下上面这段代码，`fiber.type`相当于`App` function，当我们调用它的时候会返回`h1`元素

然后，一旦有了`children`，`reconciliation`函数也能同样运作，所以我们不需要改变什么



我们还需要改造`commitWork`函数

由于存在没有`dom`节点的 fiber，我们需要改变两点

首先，`const domParent = fiber.parent.dom`这段代码不再可靠——因为函数式组件没有 dom。所以，为了找到`dom`节点的父节点，我们需要遍历 fiber tree 直到找到一个带有 dom 的 fiber

```jsx
function commitWork(fiber) {
  //...
  
	// 由于函数式组件没有 dom 的原因，所以需要不断向上遍历找到含有 dom 的 fiber
	let domParentFiber = fiber.parent;
	while (!domParentFiber.dom) {
		domParentFiber = domParentFiber.parent;
	}
	const domParent = domParentFiber.dom;
  
  //...
}
```

其次，当我们移除 dom 节点的时候，也需要递归地遍历 fiber 来找到有 dom 节点的子节点

```jsx
function commitWork(fiber) {
  //...
  else if (fiber.dom && fiber.effectTag === "DELETION") {
		commitDeletion(fiber, domParent);
	}
  //...
}

function commitDeletion(fiber, domParent) {
	if (fiber.dom) {
		domParent.removeChild(fiber.dom);
	} else {
		commitDeletion(fiber.child, domParent);
	}
}
```

