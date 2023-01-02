---
title: 深入理解 ES6 的 Class 和 extends 底层实现原理
date: 2023/1/3
updated: 2023/1/3
tags: 
- JavaScript
- ES6
categories: JavaScript
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/taylor3.jpg
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/taylor3.jpg
---
# 深入理解 ES6 的 Class 和 extends 底层实现原理

## TL，DR

ES6 的 extends 的底层实现是 **构造函数调用** 和 **寄生组合式继承** 来实现的

## 准备工作

我们首先需要准备一个 babel 的环境，通过 babel 将 ES6 的代码转译为 ES5 的代码进行阅读

推荐一个在线环境 [babel 官网的 try it out](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.20.11&externalPlugins=&assumptions=%7B%7D)

## class 的实现

首先从 `class` 的实现开始，下面这段代码涵盖了使用 `class` 时所有会出现的情况（静态属性、构造函数、箭头函数）

```javascript
class Person {
    static instance = null;
    static getInstance() {
        return super.instance;
    }
	constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    sayHi() {
        console.log('hi');
    }
    sayHello = () => {
        console.log('hello');
    }
    sayBye = function() {
        console.log('bye');
    }
}
```

而经过 babel 处理后的代码是这样的

```javascript
'use strict';

var _createClass = function () { 
    function defineProperties(target, props) { 
        for (var i = 0; i < props.length; i++) { 
            var descriptor = props[i]; 
            descriptor.enumerable = descriptor.enumerable || false; 
            descriptor.configurable = true; 
            if ("value" in descriptor) 
                descriptor.writable = true; 
            Object.defineProperty(target, descriptor.key, descriptor); 
        } 
    } 
    return function (Constructor, protoProps, staticProps) { 
        if (protoProps) 
            defineProperties(Constructor.prototype, protoProps); 
        if (staticProps) 
            defineProperties(Constructor, staticProps); 
        return Constructor; 
    }; 
}();

function _classCallCheck(instance, Constructor) { 
    if (!(instance instanceof Constructor)) { 
        throw new TypeError("Cannot call a class as a function"); 
    } 
}

var Person = function () {
  function Person(name, age) {
    _classCallCheck(this, Person);

    this.sayHello = function () {
      console.log('hello');
    };
    
    this.sayBye = function () {
      console.log('bye');
    };
    
    this.name = name;
    this.age = age;
  }

  _createClass(
    Person,
    [
      {
        key: "sayHi",
        value: function sayHi() {
          console.log("hi");
        }
      }
    ],
    [
      {
        key: "getInstance",
        value: function getInstance() {
          return _get(_getPrototypeOf(Person), "instance", this);
        }
      }
    ]
  );

  return Person;
}();

Person.instance = null;
```

最外层的 Person 变量被赋值给了一个立即执行函数，立即执行函数里面返回的是 Person 构造函数，实际上最外层的 Person 就是里面的 Person 构造函数

在 Person 类上用 `static` 定义的静态属性 `instance` 和静态方法 `getInstance()` 被直接挂载到了 Person （即 class 类）上

### 挂载属性方法

Person 类上的各个属性的关系是这样的

![](https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/class.png)

我们可以发现， Person 类上的三个方法 `sayHi`、`sayHello` 和 `sayBye`，编译后被放到了不同的地方处理

从编译后的代码中可以看到 `sayHello` 和 `sayBye` 被放到了 Person 构造函数中定义，而 `sayHi` 通过 `_createClass` 处理，被挂载到了 `Person.prototype` （即 class 类的原型）上

而在 《JavaScript》高级程序设计中提到

> 为了在实例间共享方法，类定义语法把类块中定义的方法作为**原型方法**

所以我们现在可以总结出

***因此，在 `class` 中不直接使用 `=` 来定义的方法，最终都会被挂载到原型上；使用 `=` 定义的属性和方法，最终都会被放到构造函数中。***

### _classCallCheck

Person 构造函数中调用了 `_checkCallCheck` 函数，并将 `this` 和自身传入进去。

在 `_classCallCheck` 中通过 `instanceof` 来进行判断：`instance` 是否在 `Constructor` 的原型链上面，如果不在上面则抛出错误。

这一步主要是为了避免直接将 Person 类当做函数来调用。 因此，在ES5中构造函数是可以当做普通函数来调用的，但在ES6中的类是无法直接当普通函数来调用的。

> 注意：为什么通过 `instanceof` 可以判断是否将 Person 类当函数来调用呢？ 因为如果使用 `new` 操作符实例化 Person 的时候，那么 `instance` 就是当前的实例，指向 `Person.prototype`，`instance instanceof Constructor` 必然为true。反之，直接调用 Person 构造函数，那么 instance 就不会指向 `Person.prototype`。

### _createClass

我们再来看看 `_createClass` 函数，这个函数在 Person 原型上添加了 `sayHi` 方法

```javascript
// _createClass 也是一个立即执行函数
var _createClass = function () {
  	// 将props属性挂载到目标target上面
    function defineProperties(target, props) { 
        for (var i = 0; i < props.length; i++) { 
            var descriptor = props[i]; 
            descriptor.enumerable = descriptor.enumerable || false; 
            descriptor.configurable = true; 
            if ("value" in descriptor) 
                descriptor.writable = true; 
            Object.defineProperty(target, descriptor.key, descriptor); 
        } 
    } 
  	// 这才是真正的 _createClass
    return function (Constructor, protoProps, staticProps) {
      	// 如果传入了需要挂载到原型的方法
        if (protoProps) 
            defineProperties(Constructor.prototype, protoProps);
      	// 如果传入了需要挂载到 class 类上的静态方法
        if (staticProps) 
            defineProperties(Constructor, staticProps); 
        return Constructor; 
    }; 
}();
```

`_createClass` 函数接收三个参数，分别是 `Constructor` （构造函数）、`protoProps`（需要挂载到原型的方法）、`staticProps`(需要挂载到 class 类上的静态方法)

 在接收到参数之后，`_createClass` 通过挂在函数 `defineProperties` 进行挂载，`defineProperties`对传入的 `props` 进行了遍历，并设置了其 `enumerable`（是否可枚举） 和 `configurable`（是否可配置）、`writable`（是否可修改）等数据属性。 最后使用了 `Object.defineProperty` 函数来给设置当前对象的属性描述符

## extends 实现

通过前面对 Person 的分析，相信已经了解 ES6 中类的实现，这和 ES5 中的实现大同小异，接下来我们看下 `extends` 的实现，以下面 ES6 的代码为例子

```javascript
class Child extends Parent {
  constructor(name, age) {
    super(name, age);
    this.name = name;
    this.age = age;
  }
  getName() {
    return this.name;
  }
}

class Parent {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  getName() {
    return this.name;
  }
  getAge() {
    return this.age;
  }
}
```

babel 后的代码是这样的

```javascript
"use strict";

function _inherits(subClass, superClass) {
  subClass.prototype = Object.create(superClass.prototype);
  subClass.prototype.constructor = subClass;
  _setPrototypeOf(subClass, superClass);
}
function _setPrototypeOf(o, p) {
  _setPrototypeOf = Object.setPrototypeOf
    ? Object.setPrototypeOf.bind()
    : function _setPrototypeOf(o, p) {
        o.__proto__ = p;
        return o;
      };
  return _setPrototypeOf(o, p);
}
var Child = /*#__PURE__*/ (function (_Parent) {
  _inherits(Child, _Parent);
  function Child(name, age) {
    var _this;
    _this = _Parent.call(this, name, age) || this;
    _this.name = name;
    _this.age = age;
    return _this;
  }
  return Child;
})(Parent);
var Parent = /*#__PURE__*/ (function () {
  function Parent(name, age) {
    this.name = name;
    this.age = age;
  }
  var _proto2 = Parent.prototype;
  _proto2.getName = function getName() {
    return this.name;
  };
  _proto2.getAge = function getAge() {
    return this.age;
  };
  return Parent;
})();
```

去掉一些不重要的代码，关于实现 `extends` 的核心代码如下

```javascript
var Child = /*#__PURE__*/ (function (_Parent) {
  _inherits(Child, _Parent);
  function Child(name, age) {
    var _this;
    _this = _Parent.call(this, name, age) || this;
    _this.name = name;
    _this.age = age;
    return _this;
  }
  return Child;
})(Parent);
```

可以看到这是一个立即执行函数，先调用了 `_inherits` 函数，然后在设置 `name` 和 `age` 属性之前时，使用的是执行了 `_Parent.call` 后返回的 `_this`，如果为空才使用自身的 `this`

重点分析这两步

### _inherits

先看一下 `_inherits` 的代码

```javascript
function _inherits(subClass, superClass) {
  // 将 subClass.prototype.[[prototype]] 指向 superClass.prototype
  subClass.prototype = Object.create(superClass.prototype);
  // 修正 constructor，以保证 subClass 的实例的 constructor 指向正确
  subClass.prototype.constructor = subClass;
  // 将 subClass.[[prototype]] 指向 superClass
  _setPrototypeOf(subClass, superClass);
}
function _setPrototypeOf(o, p) {
  _setPrototypeOf = Object.setPrototypeOf
    ? Object.setPrototypeOf.bind()
    : function _setPrototypeOf(o, p) {
        o.__proto__ = p;
        return o;
      };
  return _setPrototypeOf(o, p);
}
```

`_inherits` 函数接收两个参数，分别是 `subClass` （子构造函数）和 `subClass` （父构造函数），将这个函数做的事情稍微做一下梳理。

1. 设置 `subClass.prototype` 的 `[[Prototype]]`指向 `superClass.prototype`
2. 设置 `subClass` 的 `[[Prototype]]` 指向 `superClass`

这和 ES5 中的寄生组合式继承很相似，仅仅增加了第二步操作

为了方便理解，这里整理了一下原型链的关系：

![](https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/prototype.png)

###  `Object.create` 和 `Object.setPrototypeOf` 的区别

这里还涉及到一个知识点，为什么前者使用 `Object.create` 后者使用 `Object.setPrototypeOf` 

先不具体展开，只讲一下两者的区别

- `A = Object.create(B)` 会返回一个空对象，空对象的 `[[prototype]]` 指向 `B`，所以会**清空 `A` 对象**。**更偏重于重新赋值**
- `Object.setPrototypeOf(A, B)` 则是将 `A.[[prototype]]` 指向 `B`，会**保留 A 原有的内容**。**更偏重于原型链的改向**

因此，在后续修改 `subClass.[[prototype]]` 指向的时候，为了避免清空 `subClass`，采用的是 `Object.setPrototypeOf`

### _this = _Parent.call(this, name, age) || this

这一步其实就是**寄生组合式继承**里面的执行父类构造函数

那么 `Parent.call(this, name, age)` 执行后返回了什么呢？ 正常情况下，应该会返回 undefined，但不排除 `Parent` 构造函数中直接返回一个对象或者函数的可能性。 

 在构造函数中，如果什么也没有返回或者返回了原始值，那么默认会返回当前的 `this`；而如果返回的是引用类型，那么最终实例化后的实例依然是这个引用类型（仅相当于对这个引用类型进行了扩展）

## 总结

ES6 中提供的 `class` 和 `extends` 本质上只是语法糖，底层实现是 **构造函数调用** 和 **寄生组合式继承** 来实现的

所以对于一个 FrontEnd 来说，JS 的基础才是最重要的