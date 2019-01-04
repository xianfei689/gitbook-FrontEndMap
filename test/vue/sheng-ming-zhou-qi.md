# 生命周期

![](../../.gitbook/assets/image%20%283%29.png)

### 1.What is? <a id="1what-is"></a>

**Vue实例从创建到销毁的过程，就是生命周期** 。从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、销毁等一系列过程，称之为Vue的生命周期。



8 个阶段创建前/后，载入前/后，更新前/后，销毁前/后。

* 创建前/后： 在 beforeCreate 阶段，vue 实例的挂载元素 el 还没有。
* 载入前/后：在 beforeMount 阶段，vue 实例的$el 和 data 都初始化了，但还是挂载之前为虚拟的 dom 节点，data.message 还未替换。在 mounted 阶段，vue 实例挂载完成，data.message 成功渲染。
* 更新前/后：当 data 变化时，会触发 beforeUpdate 和 updated 方法。
* 销毁前/后：在执行 destroy 方法后，对 data 的改变不会再触发周期函数，说明此时 vue 实例已经解除了事件监听以及和 dom 的绑定，但是 dom 结构依然存在

