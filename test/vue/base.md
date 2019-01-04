# 双向数据绑定 MVVM

## 双向数据绑定

![](../../.gitbook/assets/image%20%283%29.png)

   **数据劫持结合发布者-订阅者** 的方式，通过 **Object.defineProperty（）** 来 **劫持** 各个属性的setter，getter，在 **数据变动时,发布消息给订阅者，触发相应监听回调。**

 vue的数据双向绑定 将MVVM作为数据绑定的入口，整合Observer，Compile和Watcher三者，通过Observer来监听自己的model的数据变化，通过Compile来解析编译模板指令（vue中是用来解析 ），最终利用watcher搭起observer和Compile之间的通信桥梁，达到数据变化 —&gt;视图更新；视图交互变化（input）—&gt;数据model变更双向绑定效果。



#### **过程解析：** <a id="&#x8FC7;&#x7A0B;&#x89E3;&#x6790;&#xFF1A;"></a>

JavaScript 对象传给 Vue 实例的 data 选项，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。每个组件实例都有相应的 watcher 实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的 setter 被调用时，会通知 watcher 重新计算，从而致使它关联的组件得以更新。



## 什么是MVVM？

 MVVM 是 Model-View-ViewModel 的缩写。

 mvvm 是一种设计思想。

 Model 层代表数据模型，也可以在 Model 中定义数据修改和操作的业务逻辑；

View 代表 UI 组件，它负责将数据模型转化成 UI 展现出来，

ViewModel 是一个同步 View 和 Model 的对象。

 在 MVVM 架构下，View 和 Model 之间并没有直接的联系，而是通过 ViewModel 进行交互，Model 和 ViewModel 之间的交互是双向的， 因此 View 数据的变化会同步到 Model 中，而 Model 数据的变化也会立即反应到 View 上。

 ViewModel 通过双向数据绑定把 View 层和 Model 层连接了起来，而 View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作 DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理。



## MVVM和MVC的区别：

 mvc 和 mvvm 其实区别并不大。都是一种设计思想。主要就是 mvc 中 Controller 演变成 mvvm 中的 viewModel。mvvm 主要解决了 mvc 中大量的 DOM 操作使页面渲染性能降低，加载速度变慢，影响用户体验。和当 Model 频繁发生变化，开发者需要主动更新到 View 。

