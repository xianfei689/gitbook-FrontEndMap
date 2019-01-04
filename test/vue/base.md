# base

## 双向数据绑定

![](../../.gitbook/assets/image%20%283%29.png)

   **数据劫持结合发布者-订阅者** 的方式，通过 **Object.defineProperty（）** 来 **劫持** 各个属性的setter，getter，在 **数据变动时,发布消息给订阅者，触发相应监听回调。**

 vue的数据双向绑定 将MVVM作为数据绑定的入口，整合Observer，Compile和Watcher三者，通过Observer来监听自己的model的数据变化，通过Compile来解析编译模板指令（vue中是用来解析 ），最终利用watcher搭起observer和Compile之间的通信桥梁，达到数据变化 —&gt;视图更新；视图交互变化（input）—&gt;数据model变更双向绑定效果。



#### **过程解析：** <a id="&#x8FC7;&#x7A0B;&#x89E3;&#x6790;&#xFF1A;"></a>

JavaScript 对象传给 Vue 实例的 data 选项，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。每个组件实例都有相应的 watcher 实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的 setter 被调用时，会通知 watcher 重新计算，从而致使它关联的组件得以更新。



