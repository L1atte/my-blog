---
title: Vue2 组件通信方式及其应用场景总结
date: 2021/10/24
updated: 2021/10/24
tags: Vue
categories: Vue
top_img: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/善逸1.png
cover: https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/善逸1.png
description: ' '
---

# Vue2 组件通信方式及其应用场景总结

## 前言

Vue框架有一大特色，就是**组件化**

即我们可以把一个复杂的页面，拆分成一个个独立的组件。再者，组件还有一个特定就是可复用性，我们可以将多个页面的共有部分抽取成一个组件，比如导航栏、底部信息、轮播图等等。

而组件实例的作用域是相互独立的，这就意味着不同组件之间的数据无法相互引用，一般来说，组件可以有以下几种关系。

![组件关系](https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/组件关系1.png)

如上图所示，A和B、B和C、B和D都是父子关系，C和D是兄弟关系，A和C、A和D是跨级组件关系。

针对不同的使用场景，如何选择行之有效的通信方式？这是我们所要探讨的主题。本文总结了vue组件间通信的几种方式。

首先我们带着这些问题去思考：

- Vue中到底有多少种组件通信方式？
- Vue中每种通信方式的应用场景是什么？
- Vue针对不同的场景，最佳通信方式是什么？

## 组件通信的8种方式

![组件通信的8种方式](https://cdn.jsdelivr.net/gh/L1atte/PicGo/img/组件的8种通信方式.png)

这是我简单总结的Vue8种组件通信方向，下面来一一进行解释。

## `props` /`$emit`

### 父组件向子组件传值 

#### 使用方法

1. 在父组件中引入子组件
2. 注册子组件
3. 在页面中使用，子组件标签上动态绑定传入动态值/静态值
4. 在子组件中，使用后`props`来接受父组件传递的值

父组件

```vue
<template>
  <div>
    <div>Father</div>
    <p></p>

    <!-- 将text1的值绑定给子组件, getMsg接受子组件的传值 -->
    <son-prop :text="text1"></son-prop>
  </div>
</template>

<script>
import SonProp from "@/components/SonProp.vue";

export default {
  name: "Father",
  components: {
    SonProp,
  },
  data() {
    return {
      text1: "text from father",
    };
  },
};
</script>
```

子组件

```vue
<template>
  <div>
    <!-- 将父组件传过来的值进行展示 -->
    <div>{{ text }}</div>
  </div>
</template>

<script>
export default {
  // 通过props接受父组件的传值，这里 text 对应父组件的 :text="text1" 中的text
  props: {
    text: { type: String },
  },
  data() {
};
</script>

<style>
</style>
```

### 子组件向父组件传递值（通过事件形式）

#### 使用方法

子组件可以使用 `$emit` 触发父组件的自定义事件，因此，我们可以通过此方法，通过`this.$emit('函数名'，传递参数)`，实现子组件向父组件传值。

父组件

```vue
<template>
  <div>
    <div>Father</div>
    <p></p>
    <!-- 接受子组件的传值 -->
    <div>{{ msg }}</div>
  </div>
</template>

<script>
import SonProp from "@/components/SonProp.vue";

export default {
  name: "Father",
  components: {
    SonProp,
  },
  data() {
    return {
      msg: "",
    };
  },
  methods: {
    getMsg(msg) {
      // 接受子组件传递来的值，赋值给data中的msg属性
      this.msg = msg;
    },
  },
};
</script>
```

子组件

```vue
<template>
  <div>
    <!-- 向父组件传值 -->
    <button @click="sendMsg">sendMsg</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msg: "text from son",
    };
  },
  methods: {
    sendMsg() {
      // 子组件通过绑定事件 this.$emit('函数名'，传递参数)
      this.$emit('sendMsg', this.msg)
    }
  }
};
</script>

<style>
</style>
```



### 特点

单向数据流。

父组件通过prop传值给子组件是属于单向数据流，因此当父组件修改该值的时候，子组件也会随之更新数据。而子组件是不应该在内部改变prop的，如果你这样做了，Vue 会在浏览器的控制台中发出警告。

### 缺点

1. prop篡改

   ​	我们在子组件使用父组件`prop`的时候，如果涉及到变量赋值、修改等操作，`prop`被莫名其妙的修改了，可能会导致父组件中的数据一同被修改。

   ​	有的人可能会疑惑，`prop`不是单向数据流吗，这里怎么会被修改呢？下面举两个例子进行证明。

   

   父组件

   ```vue
   <template>
     <div>
       <div>Father</div>
       <div>{{text1}}</div>
       
       <!-- 将text1的值绑定给子组件 -->
       <son-prop :text="text1"></son-prop>
     </div>
   </template>
   
   <script>
   import SonProp from "@/components/SonProp.vue";
   
   export default {
     name: "Father",
     components: {
       SonProp,
     },
     data() {
       return {
         text1: "text from father",
       };
     },
   };
   </script>
   ```

   子组件

   ```vue
   <template>
     <div>
       <!-- 将父组件传过来的值进行展示 -->
       <div>{{ text }}</div>
       <button @click="text='2'">改变父组件props</button>
     </div>
   </template>
   
   <script>
   export default {
     // 通过props接受父组件的传值，这里 text 对应父组件的 :text="text1" 中的text
     props: {
       text: { type: String },
     },
   };
   </script>
   
   <style>
   </style>
   ```

   当我们点击button，会抛出以下警告。并且父组件的值保持不变。

   ```
   vue.runtime.esm.js?2b0e:619 [Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value. Prop being mutated: "text"
   
   found in
   
   ---> <SonProp> at src/components/SonProp.vue
          <Father> at src/views/FatherProp.vue
            <App> at src/App.vue
              <Root>
   ```

   

   但是当我们传过来的是一个引用类型的数据，并且修改数据下的某一个属性的时候

   父组件

   ```vue
   <template>
     <div>
       <div>Father</div>
       <div>{{msg.a}}</div>
       
       <!-- 将msg的值绑定给子组件,此处msg类型为Object -->
       <son-prop :msg="msg"></son-prop>
     </div>
   </template>
   
   <script>
   import SonProp from "@/components/SonProp.vue";
   
   export default {
     name: "Father",
     components: {
       SonProp,
     },
     data() {
       return {
         msg: {
           a: 1,
         },
       };
     },
   };
   </script>
   ```

   子组件

   ```vue
   <template>
     <div>
       <!-- 将父组件传过来的值进行展示 -->
       <div>{{ msg.a }}</div>
       <!-- 点击按钮修改msg.a -->
       <button @click="msg.a='2'">change msg</button>
     </div>
   </template>
   
   <script>
   export default {
     props: {
       msg: { type: Object },
     },
   };
   </script>
   
   <style>
   </style>
   ```

   点击按钮后，并不会报错，并且父组件中的`msg.a`也相应被修改。

   借助Vue官问的一段话

   [注意在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变变更这个对象或数组本身**将会**影响到父组件的状态。](https://cn.vuejs.org/v2/guide/components-props.html#:~:text=%E6%B3%A8%E6%84%8F%E5%9C%A8%20JavaScript%20%E4%B8%AD%E5%AF%B9%E8%B1%A1%E5%92%8C%E6%95%B0%E7%BB%84%E6%98%AF%E9%80%9A%E8%BF%87%E5%BC%95%E7%94%A8%E4%BC%A0%E5%85%A5%E7%9A%84%EF%BC%8C%E6%89%80%E4%BB%A5%E5%AF%B9%E4%BA%8E%E4%B8%80%E4%B8%AA%E6%95%B0%E7%BB%84%E6%88%96%E5%AF%B9%E8%B1%A1%E7%B1%BB%E5%9E%8B%E7%9A%84%20prop%20%E6%9D%A5%E8%AF%B4%EF%BC%8C%E5%9C%A8%E5%AD%90%E7%BB%84%E4%BB%B6%E4%B8%AD%E6%94%B9%E5%8F%98%E5%8F%98%E6%9B%B4%E8%BF%99%E4%B8%AA%E5%AF%B9%E8%B1%A1%E6%88%96%E6%95%B0%E7%BB%84%E6%9C%AC%E8%BA%AB%E5%B0%86%E4%BC%9A%E5%BD%B1%E5%93%8D%E5%88%B0%E7%88%B6%E7%BB%84%E4%BB%B6%E7%9A%84%E7%8A%B6%E6%80%81%E3%80%82)

   我们可以得出结论，子组件虽然不能直接对父组件`prop`进行重新赋值，但是当父组件的传值是引用类型时，子组件可以修改父组件`prop`下的属性。

   这就是一个很尴尬的事情，如果我们设计的初衷就是父组件数据也同时被修改，这个结果是可以接受的，但是当我们不希望父组件那份数据源有任何变化的时候，这就是一个严重的逻辑`bug`。 所以这就是`props`通讯的风险项之一。

### 应用场景

父组件向子组件传值，就是正常不是嵌套很深的父子组件通信，和关系不是很复杂的兄弟组件组件通信。



## `$parent`/`$children`/`$refs`

### 使用方法

$refs: 通过在子组件上绑定 `ref`，使用 `this.$refs.refName.子组件属性/子组件方法`

$children: 获取当前实例的子组件，`$children`返回的是一个自组建的集合。如果想获取具体哪个组件的属性和方法，可以通过 `this.$children[index].子组件属性/自组件方法`

父组件

```vue
<template>
  <div>
    <!-- 给子组件绑定ref -->
    <children-1 ref="son1"></children-1>
    <children-2></children-2>
  </div>
</template>

<script>
import children1 from "../components/$children1.vue";
import children2 from "../components/$children2.vue";

export default {
  name: "father",
  components: {
    children1,
    children2,
  },
  data() {
    return {
      msg: "这是父组件的属性",
    };
  },
  methods: {
    getMsg() {
      console.log("这是父组件的方法");
    },
  },
  mounted() {
    // 通过$refs获取子组件的属性/方法
    console.log("father", this.$refs);
    // 通过$children获取子组件的属性/方法
    console.log("father", this.$children[0].getMsg);
    console.log("father", this.$children[1].msg);
  },
};
</script>

<style>
</style>
```

子组件1

```vue
<template>
  <div>
    <div>这是子组件1</div>
  </div>
</template>

<script>
export default {
  name: "son1",
  data() {
    return {
      msg: "这是子组件1的属性",
    };
  },
  methods: {
    getMsg() {
      console.log("这是子组件1的方法");
    },
  },
  created() {
    // 通过$parent获取父组件的方法
    console.log("son1",this.$parent.msg);
  },
};
</script>

<style>
</style>
```

子组件2

```vue
<template>
  <div>
    <div>这是子组件2</div>
  </div>
</template>

<script>
export default {
  name: "son2",
  data() {
    return {
      msg: "这是子组件2的属性",
    };
  },
  methods: {
    getMsg() {
      console.log("这是子组件2的方法");
    },
  },
  created() {
    // 通过$parent获取父组件的方法
    console.log("son2",this.$parent.msg);
  },
};
</script>

<style>
</style>
```

### 父组件通过 `$refs`/ `$children` 来获取子组件实例的属性和方法

```javascript
  mounted() {
    // 通过$refs获取子组件的属性/方法
    console.log("father", this.$refs);
    // 通过$children获取子组件的属性/方法
    console.log("father", this.$children[0].getMsg);
    console.log("father", this.$children[1].msg);
  },
```

注意这里在`mounted`里面才能拿到`$children`实例，写demo的时候用`created`怎么都获取不了数值，折腾了好久。具体原因与组件的生命周期有关。

### 子组件通过`$parent`获取父组件实例的属性和方法

```javascript
  created() {
    // 通过$parent获取父组件的方法
    console.log("son1",this.$parent.msg);
  },
```

### 特点

简单方便，`$refs`/`$parent`/`$children`这种通信方式，更加简单直接地获取Vue实例，对Vue实例下的数据和方法直接获取或者引用。

### 缺点

1. `this.$children`不可控性大，有一定风险。

   **需要注意 `$children` 并不保证顺序，也不是响应式的。**如果你发现自己正在尝试使用 `$children` 来进行数据绑定，考虑使用一个数组配合 `v-for` 来生成子组件，并且使用 Array 作为真正的来源。

2. 不利于组件化

   直接获取组件实例这种方式，在一定程度上妨碍了组件化开发。

   在组件化开发的过程中，在父子组件状态不透明的情况下，什么方法提供给外部，什么方法是内部使用，一切都是未知的。

   组件化开发的初衷，也不是由组件外部来对内部作出一定改变，而往往是内部的改变，通知外部绑定的方法。反过来如果是子组件内部，主动向父组件传递一些信息，也不能确定父组件是否存在。

3. 兄弟组件深层次嵌套组件通讯困难

   和props方式一样，如果是兄弟组件的通信，需要通过父组件作为中间通讯的桥梁，而深层次的组件通讯，虽然不需要像props通讯那样逐层绑定，但是有一点，需要逐渐向上层或下层获取目标实例，如何能精准获取目标实例这是一个非常麻烦的问题，而且每当深入一层，由于`$children`带来的不确定性，风险性会更大。

### 应用场景

直接通过获取实例的通信方式适合已知的、固定的页面结构，这种通讯方式，要求父子组件高度透明化，明确父子组件有哪些方法和属性，所以这种方式更适合**页面组件**，而不适合一些**第三方组件库**，或者**公共组件**。

## `$attrs`/`$listener`

### `$attrs`与`interitAttrs`之间的关系

`$attrs`: 包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

`interitAttrs`: 默认情况下父作用域的不被认作 props 的 attribute 绑定 (attribute bindings) 将会“回退”且作为普通的 HTML attribute 应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置 `inheritAttrs` 到 `false`，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例 property `$attrs` 可以让这些 attribute 生效，且可以通过 `v-bind` 显性的绑定到非根元素上

以上是Vue官方文档对于两者的解释，但是还是比较抽象的，理解起来比较困难，所以我们通过一些demo来看看具体情况。

父组件

```vue
<template>
  <div>
    <div>Father</div>
    <p></p>
    <!-- 将text1的值绑定给子组件, getMsg接受子组件的传值 -->
    <son-prop :text="text1" :msg="msg"></son-prop>
  </div>
</template>

<script>
import SonProp from "@/components/SonProp.vue";

export default {
  name: "Father",
  components: {
    SonProp,
  },
  data() {
    return {
      text1: "text from father",
      msg: "1",
    };
  },
};
</script>
```

子组件

```vue
<template>
  <div>
    <!-- 将父组件传过来的值进行展示 -->
    <div>{{ text }}</div>
  </div>
</template>

<script>
export default {
  // 通过props接受父组件的传值，这里 text 对应父组件的 :text="text1" 中的text
  props: {
    text: { type: String },
  },
};
</script>

<style>
</style>
```

父组件向子组件传递了 text 和 msg 两个变量，在子组件中，只接收了 text，并没有接收 msg，没有被接收的属性就会成为子组件根元素的属性节点`<div msg="1"></son-prop></div>`

通过这个demo，结合`$attrs`和`interitAttrs`的定义，当设置`inheritAttrs： false`时，没有被prop接收的数据都被`$attrs`实例属性接收，面包含着所有父组件传入而子组件并没有在 `props` 里显示接收的数据。

### 顶层组件通过`$attrs`向底层组件传值

通过设置选项 `inheritAttrs: false`来确保属性不会默认成为组件根元素的属性节点，同时设置`v-bing="$attrs"`向下一层组件继续传递属性。

举个例子

父组件

```vue
<template>
  <div>
    <div>Father</div>
    
    <son-prop :msg="msg"></son-prop>
  </div>
</template>

<script>
import SonProp from "@/components/SonProp.vue";

export default {
  name: "Father",
  components: {
    SonProp,
  },
  data() {
    return {
      msg: "msg",
    };
  },
};
</script>
```

子组件

```vue
<template>
  <div>
    <div>这是子组件</div>

    <grand-son v-bind="$attrs"></grand-son>
  </div>  
</template>

<script>
import GrandSon from "./GrandSon.vue";
export default {
  components: {
    GrandSon,
  },
};
</script>

<style>
</style>
```

孙子组件

```vue
<template>
  <div>
    <div>这是孙子组件</div>
    <div>{{ msg }}</div>
  </div>
</template>

<script>
export default {
  props: {
    msg: { type: String },
  },
};
</script>

<style>
</style>
```

孙子组件在 props 接收子组件中通过 `$attrs` 包裹传来的数据，同样是通过父组件传来的数据，只是在子组件用了`$attrs`进行了统一接收，再往下传递，最后通过孙子组件进行接收。

以此类推孙子组件仍然不想接收，再传入下级组件，我们仍然需要对孙子组件实例选项进行设置选项 `inheritAttrs: false`，否则仍然会成为孙子组件根元素的属性节点。

从而利用 `$attrs` 来接收 props 为接收的数据再次向下传递是一件很方便的事情。

### 顶层组件通过`$listeners`向底层组件传递方法

如何向底层组件传递属性已经了解了，面临的问题是如何向顶层组件传递数据。

`$attrs`和`$listeners`两者表面上都是一个意思，`$attrs`是向下传递属性，`$listeners`是向下传递方法。

通过手动去调用 `$listeners` 对象里的方法，原理就是 `$emit` 监听事件，`$listeners` 也可以看成一个包裹监听事件的一个对象。

父组件

```vue
<template>
  <div>
    <div>Father</div>
    <p></p>

    <!-- 接收孙子组件的传值 -->
    <div>接收孙子组件的传值： {{ msg2 }}</div>
    <son-prop @sendMsg2="getMsg2"></son-prop>
  </div>
</template>

<script>
import SonProp from "@/components/SonProp.vue";

export default {
  name: "Father",
  components: {
    SonProp,
  },
  data() {
    return {
      msg2: "msg2",
    };
  },
  methods: {
    getMsg2(msg) {
    // 接受孙子组件传递来的值，赋值给data中的msg2属性
      this.msg2 = msg;
    },
  },
};
</script>
```

子组件

```vue
<template>
  <div>
    <div>这是子组件</div>
    <!-- $listeners里存放的是父组件中绑定的非原生事件 -->
    <grand-son v-on="$listeners"></grand-son>
  </div>
</template>

<script>
import GrandSon from "./GrandSon.vue";
export default {
  components: {
    GrandSon,
  },
};
</script>

<style>
</style>
```

孙子组件

```vue
<template>
  <div>
    <div>这是孙子组件</div>
    <p></p>
    <button @click="sendMsg">向爷组件传值</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msg2: 'text from GrandSon'
    }
  },
  methods: {
    sendMsg() {
      this.$emit("sendMsg2",this.msg2)
    },
  },
};
</script>

<style>
</style>
```

子组件通过 `$listeners`将父组件的方法传递给孙子组件，然后孙子组件通过 `$emit`触发事件，向被监听的函数（v-on）的回调函数传入参数，即可实现底层组件向顶层组件传值。

### 使用场景

多层嵌套组件的情况下使用。

平时基本不会用到，但在组件颗粒度较细的场景中，以上提到的方式都不能优雅地解决诸如：A组件嵌套了B组件，B组件又嵌套了C组件，此时C要与A通信的情形（即C组件为A组件的孙子组件），Vuex会增加代码耦合度得不偿失，`$root`又局限于A组件必须是为根组件，难道只能从A传到B再从B传到C吗？是的，但`$attrs`提供了一种传递“剩余参数”的作用，使操作更简便。

## `provide` / `inject`

### 设想场景

如果说`vue`中 `provide` 和 `inject`，我会首先联想到`react`的`context`上下文,两个作用在一定程度上可以说非常相似,在父组件上通过`provide`将方法，属性，或者是自身实例暴露出去，子孙组件，插槽组件,甚至是子孙组件的插槽组件，通过`inject`把父辈`provide`引进来。提供给自己使用，很经典的应用 `provide`和 `inject`的案例就是 `element-ui`中 `el-form`和 `el-form-item`

我们试着想象一个场景

```html
<el-form  label-width="80px" :model="formData">
  <el-form-item label="姓名">
    <el-input v-model="formData.name"></el-input>
  </el-form-item>
  <el-form-item label="年龄">
    <el-input v-model="formData.age"></el-input>
  </el-form-item>
</el-form>
```

我们可以看到 `el-form` 与 `el-form-item`不需要建立任何通信操作，那么`el-form` 与 `el-form-item`是如何联系起来，并且共享状态的呢？我们带着问题继续往下看。

### 基本使用方法

我们用父组件 → 子组件 → 孙组件的案例

父组件

```vue
<template>
  <div>
    <div>这是父组件</div>
    <div>子组件对我说： {{ sonMsg }}</div>
    <div>孙子组件对我说： {{ grandSonMsg }}</div>
    <son></son>
  </div>
</template>

<script>
import Son from './Son.vue';
export default {
  name: 'father',
  components: {
    Son,
  },
  provide() {
    return {
      // 将自己暴露给子孙组件，这里声明的名称要于子组件引进的名称保持一致
      father: this
    }
  },
  data() {
    return {
      sonMsg: '', // 来自子组件的信息
      grandSonMsg: '', // 来自孙组件的信息
    }
  },
     methods:{
      /* 接受孙组件信息 */
      grandSonSay(value){
          this.grandSonMsg = value
      },
      /* 接受子组件信息 */ 
      sonSay(value){
          this.sonMsg = value
      },
   },
};
</script>

<style>
</style>
```

子组件

```vue
<template>
  <div>
    <div>这是子组件</div>
    <p></p>
    <input v-model="msg" />
    <button @click="send">对父组件说</button>
    <p></p>
    <grand-son></grand-son>
  </div>
</template>

<script>
import GrandSon from './GrandSon.vue';
export default {
  name: "son",
  components: {
    GrandSon,
  },
  // 引入父组件
  inject: ["father"],
  data() {
    return {
      msg: "",
    };
  },
  methods: {
    send() {
      this.father.sonSay(this.msg);
    },
  },
};
</script>

<style>
</style>
```

孙组件

```vue
<template>
  <div>
    <div>这是孙组件</div>
    <p></p>
    <input v-model="msg" />
    <button @click="send">对爷组件说</button>
  </div>
</template>

<script>
export default {
  name: "grandSon",
  inject: ["father"],
  data() {
    return {
      msg: "",
    };
  },
  methods: {
    send() {
      this.father.grandSonSay(this.msg);
    },
  },
};
</script>

<style>
</style>
```

子孙组件通过`inject`把父组件实例引进来，然后可以直接通过`this.father`可以直接获取到父组件，并调用下面的`sonSay`方法。

### 插槽使用方法

父组件

```vue
<template>
  <div>
    <div>这是父组件</div>
    <div>子组件对我说： {{ sonMsg }}</div>
    <div>孙子组件对我说： {{ grandSonMsg }}</div>
    <son>
      <grand-son></grand-son>
    </son>
  </div>
</template>

<script>
import GrandSon from './GrandSon.vue';
import Son from './Son.vue';
export default {
  name: 'father',
  components: {
    Son,
    GrandSon,
  },
  provide() {
    return {
      // 将自己暴露给子孙组件，这里声明的名称要于子组件引进的名称保持一致
      father: this
    }
  },
  data() {
    return {
      sonMsg: '', // 来自子组件的信息
      grandSonMsg: '', // 来自孙组件的信息
    }
  },
     methods:{
      /* 接受孙组件信息 */
      grandSonSay(value){
          this.grandSonMsg = value
      },
      /* 接受子组件信息 */ 
      sonSay(value){
          this.sonMsg = value
      },
   },
};
</script>

<style>
</style>
```

子组件

```vue
<template>
  <div>
    <div>这是子组件</div>
    <p></p>
    <input v-model="msg" />
    <button @click="send">对父组件说</button>
    <p></p>
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "son",

  // 引入父组件
  inject: ["father"],
  data() {
    return {
      msg: "",
    };
  },
  methods: {
    send() {
      this.father.sonSay(this.msg);
    },
  },
};
</script>
```

达到同样的效果。实际上这种插槽方式，所有在父组件注册的组件，最后孙组件也会绑定到子组件的slot上，和上述情况差不多。

### 优点

1. `provide`/`inject`不受子组件层级的影响

2. 适用于插槽，嵌套插槽

   `provide inject` 让插槽嵌套的父子组件通信变得简单，这就是刚开始我们说的，为什么 `el-form` 和 `el-form-item`能够协调管理表单的状态一样。在`element`源码中 `el-form` 就是将`this`本身`provide`出去的。

### 缺点

1. 不适合兄弟通讯

   `provide-inject` 协调作用就是获取父级组件们提供的状态，方法，属性等，流向一直都是由父到子，`provide`提供内容不可能被兄弟组件获取到的，所以兄弟组件的通信不肯能靠这种方式来完成。

2. 父级组件无法主动通信

   `provide-inject`更像父亲挣钱给儿子花一样，儿子可以从父亲这里拿到提供的条件，但是父亲却无法向儿子索取任何东西。正如这个比方，父组件对子组件的状态一无所知。也不能主动向子组件发起通信。

### 应用场景

`provide-inject`这种通信方式，更适合深层次的复杂的父子代通信，子孙组件可以共享父组件的状态，还有一点就是适合`el-form` `el-form-item`这种插槽类型的情景。

## Vuex

## 事件总线—— EventBus

## 事件总线二—— new Vue

