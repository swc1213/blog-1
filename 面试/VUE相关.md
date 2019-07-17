# Vue面试题

## vue-router 怎么实现路由切换

分为几步：

1. 调用router.push函数，在函数中调用history.push函数，执行transitionTo函数
   1. 根据当前路径和目标路径去获取匹配到的路径
   2. 拿着匹配到的路径执行路径切换，这里执行的是一个异步，因为可能会有异步加载的组件。
      1. 根据匹配当前路径的routes和目标路径匹配的routes解析出3个数组，分别是要执行更新操作的路径数组，执行失活操作的路径数组和执行激活操作的路径数组。
      2. 然后执行导航守卫的方法，执行过程是构造一个队列数组，然后定义一个迭代器，最后执行这个队列（依次执行），迭代器就是执行每一个导航守卫，并传入目标route，当前route和匿名函数，对应to,from,next，所以只有执行了next才会继续往下走。
      3. 构造的队列数组顺序：
         1. 在失活的组件中调用离开守卫（beforeRouteLeave）
         2. 调用全局的beforeEach守卫
         3. 在更新的组件中调用beforeRouteUpdate守卫
         4. 在激活的路由中调用beforeEach
         5. 解析异步路由组件，获取所有被激活的组件
         6. 在被激活的组件里调用beforeRouteEnter，在这里注意，此钩子拿不到组件实例，因为组件还没有被创建，但是在这里可以通过next拿到组件实例
            1. 为啥next里能拿到？是因为在绑定时把next回调收集到callbacks里面了，在最后nexttick里执行cbs。这时候就能拿到组件实例了。
         7. 调用全局的beforeResolve
         8. 执行onComplete之后，执行updateRoute函数，此时调用全局的afterEach
2. 在transitionTo的回调中，执行修改url，调用pushHash函数，修改浏览器地址
   1. 先检查是不是支持pushState，不支持的话直接改浏览器地址，支持的话就执行pushState方法，调用原生history的pushState或者replaceState方法更新地址，并把历史放到历史栈里。
   2. 在history初始化时设置一个监听器监听历史栈的变化，当点击浏览器返回按钮时，如果已经有url被放到历史栈里，就出发popstate方法，拿到当前路径，目标路径，重新走一边transitTo方法。
3. 组件渲染。每次切换路由，会触发app._route的setter事件，通知router-view的watcher更新，重新渲染组件。
   1. 获取当前路径，然后遍历查找嵌套router-view，找到所有需要渲染的组件。
   2. 定义注册路由实例的方法。
   3. 根据component渲染出对应组件vnode
4. 操作滚动条位置

## VUE里面的a标签和router-link标签有什么区别

不管是HTML5 history模式还是hash模式，router-link的表现都一致
在history模式下，router-link会守卫点击事件，让浏览器不会重新加载页面。
history模式下使用base字段后，to属性无需再写base路径了。

## a标签的默认事件被禁掉以后做了什么才实现了路由跳转

执行了push函数，走上面那一套流程实现跳转。

## Vue性能提升

1. 使用生产环境
2. 使用单文件模板，CSS抽取到css文件中
3. 利用object.freeze()提升性能
4. 扁平化store数据结构
5. 合理使用持久化Store数据
6. 组件懒加载
7. 服务端渲染/预渲染（SSR）

## 前端性能提升大致分几类

1. 代码压缩，优化执行顺序
2. 尽量减少CSS全局设置的变动
3. 依赖代码拆开引入
4. 代码内部注意编码。

## import {Button} from 'antd', 打包的时候只打包button，分模块加载是怎么做到的

首先实现方式是commonJS的解构，webpack里存放了所有组件的资源位置，当只需要一种的时候，webpack自动转换成 var button = require('components/lib/button.js');require('components/lib/button/style.css')，从而实现分模块打包

## 使用import时，webpack对node_modules里的依赖会做什么

在 Node.js 模块系统中，如果 require 的模块不是核心模块，而且没有 './' 之类的开头，那就需要从当前 package 的 node_modules 里面找，找不到就到当前 package 目录上层 node_modules 里面取... 一直找到全局 node_modules 目录。这样找到的往往是文件夹，所以接下来就是处理一个文件目录作为 Node 模块的情况。如果文件目录下有 package.json，就根据它的 main 字段找到 js 文件。如果没有 package.json，那就默认取文件夹下的 index.js。由于 webpack browsersify 等模块打包工具是兼容 node 的模块系统的，自然也会进行同样的处理流程。不同的是，它们支持更灵活的配置。比如在 webpack 里面，可以通过 alias 和 external 字段配置，实现对默认 import 逻辑的自定义。

## vue 的双向绑定的原理是什么

vue.js 是采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。
　　具体步骤：
　　第一步：需要 observe 的数据对象进行递归遍历，包括子属性对象的属性，都加上 setter 和 getter
这样的话，给这个对象的某个值赋值，就会触发setter，那么就能监听到了数据变化
　　第二步：compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图
　　第三步：Watcher订阅者是Observer和Compile之间通信的桥梁，主要做的事情是:
　　1、在自身实例化时往属性订阅器(dep)里面添加自己
　　2、自身必须有一个update()方法
　　3、待属性变动dep.notice()通知时，能调用自身的 update() 方法，并触发Compile中绑定的回调，则功成身退。
　　第四步：MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更的双向绑定效果

## 简述vue中的DOM DIFF算法

当数据发生改变时，set方法会让调用Dep.notify通知所有订阅者Watcher，订阅者就会调用patch给真实的DOM打补丁(两个重要函数patchVnode和updateChildren)：

* 先判断根结点及变化后的节点是否是sameVnode，如果不是的化，就会创建新的根结点并进行替换
* 如果是sameVnode，则进入patchVnode函数，其基本判断
  1. 如果两个节点是相等oldVnode === vnode则直接return
  2. 如果新节点是文本节点，则判断新旧文本节点是否一致，不一致(oldVnode.text !== vnode.text)则替换
  3. 如果新节点不是文本节点，则开始比较新旧节点的子节点oldCh和ch
  4. 如果子节点都存在，则进行updateChildren计算
  5. 如果只有新子节点存在,则如果旧节点有文本节点，则移除文本节点，然后将新子节点拆入
  6. 如果只有旧子节点存在，则移除所有子节点
  7. 如果均无子节点且旧节点是文本节点，则移除文本节点（此时新节点一定不是文本节点）

## webpack 生命周期，loader和plugin区别

对于loader，它就是一个转换器，将A文件进行编译形成B文件，这里操作的是文件，比如将A.scss或A.less转变为B.css，单纯的文件转换过程
对于plugin，它就是一个扩展器，它丰富了wepack本身，针对是loader结束后，webpack打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听webpack打包过程中的某些节点
run：开始编译
make：从entry开始递归分析依赖并对依赖进行build
build-moodule：使用loader加载文件并build模块
normal-module-loader：对loader加载的文件用acorn编译，生成抽象语法树AST
program：开始对AST进行遍历，当遇到require时触发call require事件
seal：所有依赖build完成，开始对chunk进行优化（抽取公共模块、加hash等）
optimize-chunk-assets：压缩代码
emit：把各个chunk输出到结果文件
通过对节点的监听，从而找到合适的节点对文件做适当的处理

## Vuex相关

## 介绍AST抽象语法树

抽象语法树是源代码的抽象语法结构的树状表示，里面每个节点都是源码中的一种结构。

## timeout interval promise 队列顺序

node是观察者模式实现事件循环，异步调用设置好回调和参数后将方法放入线程池，事件循环从IO观察者取到可用的请求对象，就从线程池里拿一个函数过来执行，执行完成后把IO交还给IO观察者，从而实现事件循环。
在这里介绍非异步IO：setTimeout, setInterval, setImmediate, process.nextTick.

setTimeout和setInterval不会放入线程池，会放到定时器观察者内部的一个红黑树里，每次Tick执行时会从红黑树中取出定时器对象，检查时间是否超过，如果超过了就立即执行回调函数。但定时器并非精确，如果遇到事件循环时间长，可能在下一次循环前已经超时了。

process.nextTick则是把回调放到队列中，下次tick就执行，和setImmediate很像。但是nextTick会比setImmediate要更优先，因为nextTick属于idea观察者，setImmediate属于check观察者，idea优先级高于IO，IO优先级高于check。

在具体实现上，process.nextTick 的回调存在数组中，setImmediate()的结果是保存在链表上，这样就会导致process.nextTick会在每轮循环中把数组中所有回调全部执行。而setImmediate则会在每轮循环中执行链表中的一个回调。

promise.Trick()>promise的回调>setTimeout>setImmediate

## new Vue 发生了什么

首先new是实例化一个对象，Vue实际上是一个类，在类的实例化过程中，做了：
    合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化 data、props、computed、watcher 等等。

vm._render 最终是通过执行 createElement 方法并返回的是 vnode，它是一个虚拟 Node
