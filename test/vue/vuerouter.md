# VueRouter

## 嵌套路由怎么定义：

 VueRouter 的参数中使用 children 配置，这样就可以很好的实现路由嵌套。

index.html 只有一个路由出口

```javascript
<div id="app">
    <!-- router-view 路由出口, 路由匹配到的组件将渲染在这里 -->
    <router-view></router-view>
</div>
```

 main.js，路由的重定向，就会在页面一加载的时候，就会将 home 组件显示出来，因为重定向指向了 home 组件，redirect 的指向与 path 的必须一致。children 里面是子路由，当然子路由里面还可以继续嵌套子路由。

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)
//引入两个组件
import home from "./home.vue"
import game from "./game.vue"
//定义路由
const routes = [
    { path: "/", redirect: "/home" },//重定向,指向了home组件
    {
        path: "/home", component: home,
        children: [
            { path: "/home/game", component: game }
        ]
    }
]
//创建路由实例
const router = new VueRouter({routes})
new Vue({
    el: '#app',
    data: {
    },
    methods: {
    },
    router
})
```

 home.vue，点击显示就会将子路由显示在出来，子路由的出口必须在父路由里面，否则子路由无法显示。

