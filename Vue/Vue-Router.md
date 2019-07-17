# Vue-Router

## Vue插件注册原理

Vue 通过 Vue.use的全局API来注册插件，实现原理：  
Vue.use 接受一个 plugin 参数，并且维护了一个 _installedPlugins 数组，它存储所有注册过的 plugin；接着又会判断 plugin 有没有定义 install 方法，如果有的话则调用该方法，并且该方法执行的第一个参数是 Vue；最后把 plugin 存储到 installedPlugins 中。

## Vue-Router注册

当用户执行Vue.use(VueRouter)时，实际上就是在执行install函数，在Vue-Router文件里定义了一个全局变量_Vue，用于存储Vue对象，免去请求依赖文件。然后在Vue.mixin中把beforeCreate和destroyed钩子函数注入到每一个组件中。  
在beforeCteated中，定义了 this._routerRoot 表示它自身；this._router 表示 VueRouter 的实例 router，它是在 new Vue 的时候传入的；另外执行了 this._router.init() 方法初始化 router，然后用 defineReactive 方法把 this._route 变成响应式对象。  
接着给 Vue 原型上定义了 $router 和 $route 2 个属性的 get 方法，这就是为什么我们可以在组件实例上可以访问 this.$router 以及 this.$route，它们的作用之后介绍。 
接着又通过 Vue.component 方法定义了全局的 <router-link> 和 <router-view> 2 个组件，这也是为什么我们在写模板的时候可以使用这两个标签。

## Vue-Router对象

Vue-Router的实现是一个类，当我们执行new VueRouter()的时候做了哪些事情？
构造函数定义了一些属性，其中 this.app 表示根 Vue 实例，this.apps 保存持有 $options.router 属性的 Vue 实例，this.options 保存传入的路由配置，this.beforeHooks、 this.resolveHooks、this.afterHooks 表示一些钩子函数。this.matcher 表示路由匹配器，this.fallback 表示在浏览器不支持 history.pushState 的情况下，根据传入的 fallback 配置参数，决定是否回退到hash模式，this.mode 表示路由创建的模式，this.history 表示路由历史的具体的实现实例，它是根据 this.mode 的不同实现不同，它有 History 基类，然后不同的 history 实现都是继承 History。  
实例化 VueRouter 后会返回它的实例 router，我们在 new Vue 的时候会把 router 作为配置的属性传入，然后在插件注册时的beforeCreate的时候会执行init方法。

