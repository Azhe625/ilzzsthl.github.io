---
title: JavaScript模块化的理解
date: 2020-01-16 15:21:58
tags:
    - 前端
    - JavaScript
categories: 前端
---
## 前言
在开发时经常会听到模块化，比如说<font color="#c7254e">CommonJS、AMD、CMD、ES6</font>模块化，但一直没有去了解这些到底是个啥，有什么用。今天在分析源码时就遇到了这些模块化的语法，因此认为还是有必要去了解一番的，在此做个总结。

疑问四部曲：什么是模块，什么时候使用模块化，模块化用在哪？解决了什么问题？  
理解：模块就是将一个大文件拆分成多个独立且相互依赖的小模块，各个模块可以通过特定语法去引入其他模块，在代码量多、项目逐渐庞大时使用模块化，解决了以下问题：  
1. 利于维护，通过引入方式我们可以直观的了解依赖关系
2. 多人开发时只负责自己独立的一块模块，不需要关心别人写的代码
3. 不会造成命名冲突
4. ...  

## CommonJS规范
> CommonJS 是以在浏览器环境之外构建 JavaScript 生态系统为目标而产生的项目，比如在服务器和桌面环境中。    
>CommonJS 规范是为了解决 JavaScript 的作用域问题而定义的模块形式，可以使每个模块它自身的命名空间中执行。该规范的主要内容是，模块必须通过 module.exports 导出对外的变量或接口，通过 require() 来导入其他模块的输出到当前模块作用域中。  

Node.js开发者一定会对CommonJS眼熟，因为Node开发就是采用的CommonJS规范，demo如下：
```javascript
// moduleA.js
module.exports = function( value ){
    return value * 2;
}

// moduleB.js
var multiplyBy2 = require('./moduleA');
var result = multiplyBy2(4);
```
上图的moduleA.js通过<font color="#c7254e">module.exports</font>导出了一个函数，moduleB.js通过<font color="#c7254e">require()</font>去获取了moduleA这个模块，调用了此函数  
CommonJS加载模块是同步的，虽然是为了服务器和桌面环境产生的规范，但是也有浏览器端的实现。

## AMD规范
>AMD（异步模块定义）是为浏览器环境设计的，定义了一套 JavaScript 模块依赖异步加载标准，因为 CommonJS 模块系统是同步加载的，当前浏览器环境还没有准备好同步加载模块的条件。

由于CommonJS是为了服务器和桌面设计的规范，是同步的，那么AMD的不同处就是为了浏览器环境设计的，是异步的，requireJS是基于AMD规范的模块化库。  
模块通过<font color="#c7254e">define</font>函数定义在闭包中
```javascript
    define(id?: String, dependencies?: String[], factory: Function|Object);
```
<font color="#c7254e">id</font>是可选的，代表模块名。  
<font color="#c7254e">dependencies</font>也是可选的，代表指定所要依赖的模块列表，默认值是["require", "exports", "module"]  
<font color="#c7254e">factory</font>是一个函数或对象，包裹的是模块的具体实现，如果是函数，那么它的返回值就是模块的输出接口或值。  
光看文字理解起来比较复杂，我们来看看demo帮助理解:
```javascript
//定义一个名字为myModule的模块，依赖于jQuery模块
define('myModule', ['jquery'], function($) {
    // $ 是 jquery 模块的输出
    $('body').text('hello world');
});
//使用上面的模块
require(['myModule'], function(myModule) {});
```
我们在使用myModule这个模块时，首先加载了jquery模块，并使用了jquery的语法在页面中输出了一句话，然后再触发回调function，去继续运行function内的代码，由此可见是一个异步的方式

## CMD规范
CMD与AMD相近，都是异步加载，用于浏览器环境中，seaJS是基于CMD规范的模块化库。  
不同之处是
1. AMD的模块是提前执行的，CMD是延迟执行的。
2. AMD推崇定义模块时就要声明依赖的模块(依赖前置)，CMD推崇只有用到某一个模块时再去加载模块(依赖就近)  
demo如下:
```javascript
//AMD
define(['./a','./b'], function (a, b) {
    //可以直接使用依赖方法
    a.test();
    b.test();
});

//CMD
define(function (requie, exports, module) {
    //需要使用的时候 在require
    var a = require('./a');
    a.test();
});
```
  
## ES6模块化
ES6已经自带了模块化，而且使用方法更为简单，主要使用的的语法是<font color="#c7254e">export</font>和<font color="#c7254e">import</font>。demo如下:
### export/import 导入导出某变量/函数
1. 对变量的导入导出
```javascript
    //第一种导出
    export var name = 'lz'
    //第二种导出
    var name = 'lz'
    export { name }
    //导出的时候重命名
    export {name as newName}

    //导入
    import { name } from './test.js'
    //为变量重新定义名称导入
    import { name as newName } from './test.js'
```
2. 对函数的导出导入
```javascript
    //第一种导出
    export function sayHi() {}
    //第二种导出
    function sayHi() {}
    export { sayHi }
    //导出的时候重命名
    export { sayHi as newFunction }

    //导入
    import { sayHi } from './test.js'
    //为变量重新定义名称导入
    import { sayHi as newFunction } from './test.js'
```
3. 对变量函数的整体导入  
之前的例子是针对某一个名称去导入，也可以整体导入
```javascript
    import * as test from './test.js
```
所有的输出值都加载到test这个对象上面，使用时就直接test.××。  
注意一点，模块整体导入的对象是不允许运行时改变的
```javascript
    //两者都是不允许的
    test.name = 'lz'
    test.sayHi = function() {}
```  

### export defaut的使用
之前都是已知函数名、变量名时进行的导入，当然，不需要知道变量/函数名也是可以导入导出。此处用到了<font color="#c7254e">export defaut</font>方法，为模块指定默认的输出。  
1. 匿名函数的导出导入
```javascript
    //第一种导出
    export default function() {}
    //导入
    import sayHi from './test.js'
    sayHi()
```
2. 非匿名函数的导出导出
```javascript
    export default function sayHi() {}
    //或者
    function sayHi() {}
    export default sayHi

    //上面代码定义的函数名，在模块外部加载是无效的，视为匿名函数
    import sayHi from './test.js'
    //或
    import hello from './test'
```
3. default的本质
default的本质其实就是输出一个叫做default的变量或方法，然后允许为它取任何名字。
```javascript
    function sayHi(){}
    export { sayHi as default}
    //等价于
    export default sayHi

    import { default as sayHi } from './test.js'
    //等价于
    import sayHi from './test.js'
```
所以看出default相当于输出了一个叫default的变量，以下是错误的声明
```javascript
    //已经声明了一次 
    export default var a = 1
    //未声明，没有指定对外的接口
    export 250
```
4. export default和export同时使用
```javascript
    //导出
    export default sayHi() {}
    function hello() {

    }
    export { hello as helloWorld }
    //导入
    import sayHi,{ hello, helloWorld } from './test.js'
```
上面的hello和helloWorld都指向同一个方法，暴露出helloWorld，默认指向hello接口
5. export 也可以输出类
```javascript
    export defalut class{ ... }
```
以上就是ES6模块化的基本使用方法。当然ES6的模块化不止这些，比如export和import复合写法、模块继承、跨模块常量等等，这些内容需要深入的学习ES6知识。
## 总结
前端模块化的内容暂时整理了这么多，后续学到会继续更新。从只听说过模块化，到现在能区分出各个模块的规范及使用方法，这次总结对我来言确实学习到了模块化方面的知识，希望对大家也能有所帮助。

## 参考链接
[阮一峰-ES6入门](http://es6.ruanyifeng.com/#docs/module#export-default-%E5%91%BD%E4%BB%A4)  
[CommonJS规范和AMD规范](https://zhaoda.net/webpack-handbook/commonjs.html)