# JS基础

想到哪写到哪

### 理解proto和prototype
函数的__proto__ 指向Function的prototype

### new 运算符是如何工作的

* 创建类实例，把实例对象的__proto__ 属性对应到类的prototype上
* 初始化实例，类被传入参数并调用，this设置成实例
* 返回该实例

### 关于 this

全局环境下，this指向window，'use strict'严格模式下，this指向undefined  
任何环境下，声明变量时没有使用var或let，则变量被挂载到window上  
node环境中，全局this指向global，node执行js脚本的时候，this指向空对象  
函数正常被调用时，里面的this指向window，还是严格模式除外  
调用函数时使用了new关键字，此时this指向新的上下文，这个上下文可以理解为实例的对象体内吧。  
把实例的方法作为参数传递时，实例是不会跟着过去的。也就是说，此时方法中的this在调用时指向的是全局this或者是undefined在声明了"use strict"时。解决方法就是传递的时候使用bind方法显示指明上下文，bind方法是所有函数或方法都具有的。  
