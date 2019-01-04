# VueX

\*\*\*\*

\*\*\*\*

**1、vuex有哪几种属性？**  
答：有五种，分别是 State、 Getter、Mutation 、Action、 Module  
  
  
**2、vuex的State特性是？**  
答：  
一、Vuex就是一个仓库，仓库里面放了很多对象。其中state就是数据源存放地，对应于与一般Vue对象里面的data  
二、state里面存放的数据是响应式的，Vue组件从store中读取数据，若是store中的数据发生改变，依赖这个数据的组件也会发生更新  
三、它通过mapState把全局的 state 和 getters 映射到当前组件的 computed 计算属性中  
  
  
**3、vuex的Getter特性是？**  
答：  
一、getters 可以对State进行计算操作，它就是Store的计算属性  
二、 虽然在组件内也可以做计算属性，但是getters 可以在多组件之间复用  
三、 如果一个状态只在一个组件内使用，是可以不用getters  
  
  
**4、vuex的Mutation特性是？**  
答：  
一、Action 类似于 mutation，不同在于：  
二、Action 提交的是 mutation，而不是直接变更状态。  
三、Action 可以包含任意异步操作  
  
  
  
**5、Vue.js中ajax请求代码应该写在组件的methods中还是vuex的actions中？**  
答：  
一、如果请求来的数据是不是要被其他组件公用，仅仅在请求的组件内使用，就不需要放入vuex 的state里。  
二、如果被其他地方复用，这个很大几率上是需要的，如果需要，请将请求放入action里，方便复用，并包装成promise返回，在调用处用async await处理返回的数据。如果不要复用这个请求，那么直接写在vue文件里很方便。  
  
  
**6、不用Vuex会带来什么问题？**  
答：

一、可维护性会下降，你要想修改数据，你得维护三个地方

二、可读性会下降，因为一个组件里的数据，你根本就看不出来是从哪来的

三、增加耦合，大量的上传派发，会让耦合性大大的增加，本来Vue用Component就是为了减少耦合，现在这么用，和组件化的初衷相背。  
  
**但兄弟组件有大量通信的，建议一定要用**，不管大项目和小项目，因为这样会省很多事

