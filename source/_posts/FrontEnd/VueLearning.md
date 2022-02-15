# Vue学习记录

## Vue语法学习

> 这里记录一些Vue的语法相关的一些理解

### 模板语法

对于插值主要包含两个部分：

* 文本插值，通过双大括号（Mustache）进行数据绑定。将对应的文本值绑定为对应组件实例的属性（property）

  ```html
  <span>Message: {{ msg }}</span>
  ```

* 属性插值。这里不再能使用Mustache进行数据绑定，需要通过使用v-bind的语法

  ```html
  <div v-bind:id="dynamicId"></div>
  ```

> HTML元素的一些理解：
>
> * 本质每个HTML元素都是一些**文本+属性**的构成，至于不同的标签只是相当于一些内定的样式。同时HTML元素是可以相互嵌套的。（BTW：感觉这样一来，本质上网页的大部分元素都是可以通过div元素+相应样式设定来构成。<input>类似的应该不太行？）
> * 文本：给出了需要显示的内容。 属性：对于文本进行一些设定（如样式style设定，归属的类class设定等等

上述文本、属性的数据绑定都是支持**JS表达式**

### v-语法

* v-bind用于html元素属性的模板值绑定

  * 基本用法：`v-bind:key="attributeName"`以及`:key="attributeName"`

* v-if用于条件渲染，根据属性的bool值决定元素的可见性

* v-for用于遍历当前组件实例的属性（类型可以是数组、对象）

  * 这里使用Key属性的 原因和原理没看懂

    ```html
    <div v-for="item in items" :key="item.id">
      <!-- 内容 -->
    </div>
    ```

* v-on用于绑定html元素监听事件

  * 基本用法：`v-on:click="methodName"`以及`@click="methodName"`
  * 这里事件修饰符当中的stop函数等看不懂

* v-model用于输入数据的反向绑定

### 组件基础

* 创建一个Vue应用需要一个根组件

* 根组件和其他组件本质上是一样的，只是因为其层级最高 称为根组件

* v-bind同样可以设定自定义组件的属性

* v-on可以监听自定义组件的事件（这里事件需要自定义组件通过使用`$emit`的方式进行发出

* 实现页面的动态切换：

  * 通过component的is属性来进行切换 基本思路如下：通过component所属组件的某个属性进行控制（下面的例子就是通过currentTabComponent属性进行控制） 

    ```html
    <!-- 组件会在 `currentTabComponent` 改变时改变 -->
    <component :is="currentTabComponent"></component>
    ```

* 注意组件template中变量名**横划线**和**驼峰**的问题

### Vue路由

> 这里的路由需要注意区分一下和后端路由不同，这里是前端路由（即不会向服务器重新请求HTML、JS、CSS等文件数据）

```js
// 这里为什么不是import？？？？
const { createApp, h } = Vue

// 这里定义 需要渲染的组件对象
const NotFoundComponent = { template: '<p>Page not found</p>' }
const HomeComponent = { template: '<p>Home page</p>' }
const AboutComponent = { template: '<p>About page</p>' }

// 这里利用对象的k-v将路由路径和上述组件对象对应起来了
const routes = {
  '/': HomeComponent,
  '/about': AboutComponent
}

// 这里在定义一个路由组件对象（这里该组件会根据当前的路由路径进行切换）
// 也是后续的根组件
const SimpleRouter = {
  // 这里定义了一个data的命名函数
  data: () => ({
    currentRoute: window.location.pathname
  }),
  /** 这边是更常见的写法，使用了ES6中的对象 函数简洁定义的方式
  data () {
   currentRoute: window.location.pathname
 },
  **/

  // 这里使用了Vue的计算属性computed
  computed: {
    CurrentComponent() {
      return routes[this.currentRoute] || NotFoundComponent
    }
  },
 
  // render函数这里返回需要渲染的组件内容
  // 优先级render > templates > 所挂载元素中内容
  render() {
    return h(this.CurrentComponent)
  }
}

createApp(SimpleRouter).mount('#app')
```

### 小结

[Vue官方文档-基础部分](https://v3.cn.vuejs.org/guide/installation.html)给出了Vue框架的一些上述的基础使用。

几个特点：

* 通过将整个页面划分成树状的组件进行构建
* 组件中template纯粹负责视图部分，script部分重点在于关心数据逻辑交互方面的问题
* Vue框架背后完成了和DOM元素的交互，而且是响应式（这里我的理解是不同元素之间被判断之间的依赖关系，其中一个元素发生变化后，其余元素自动响应渲染）

## Vue应用部署

> 想要了解的一些内容：
>
> * [ ] Vue应用一般构建过程使用的工具链是啥
> * [ ] Webpack是干啥的
> * [ ] 这些工具背后实现了什么操作



## Vue实战

>* [ ] 实现一个2048游戏
>* [ ] 实现师门内部监控的交互页面（包括一些博客内容以及线上PPP）等等