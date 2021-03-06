# 生命周期

![](http://zhouxianfei.gitee.io/imgstore/front/frontEndMap/7.0.png)

## 1.What is?

**Vue实例从创建到销毁的过程，就是生命周期** 。从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、销毁等一系列过程，称之为Vue的生命周期。

8 个阶段创建前/后，载入前/后，更新前/后，销毁前/后。

* 创建前/后： 在 beforeCreate 阶段，vue 实例的挂载元素 el 还没有。
* 载入前/后：在 beforeMount 阶段，vue 实例的$el 和 data 都初始化了，但还是挂载之前为虚拟的 dom 节点，data.message 还未替换。在 mounted 阶段，vue 实例挂载完成，data.message 成功渲染。
* 更新前/后：当 data 变化时，会触发 beforeUpdate 和 updated 方法。
* 销毁前/后：在执行 destroy 方法后，对 data 的改变不会再触发周期函数，说明此时 vue 实例已经解除了事件监听以及和 dom 的绑定，但是 dom 结构依然存在

**1、什么是vue生命周期？**

答： Vue 实例从创建到销毁的过程，就是生命周期。从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、销毁等一系列过程，称之为 Vue 的生命周期。

**2、vue生命周期的作用是什么？**

答：它的生命周期中有多个事件钩子，让我们在控制整个Vue实例的过程时更容易形成好的逻辑。

**3、vue生命周期总共有几个阶段？**

答：它可以总共分为8个阶段：创建前/后、载入前/后、更新前/后、销毁前/销毁后。

**4、第一次页面加载会触发哪几个钩子？**

答：会触发下面这几个beforeCreate、created、beforeMount、mounted 。

**5、DOM 渲染在哪个周期中就已经完成？**

答：DOM 渲染在 `mounted` 中就已经完成了。

