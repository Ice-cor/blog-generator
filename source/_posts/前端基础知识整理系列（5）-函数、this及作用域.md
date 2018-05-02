---
title: '前端基础知识整理系列（5）: 函数及this'
date: 2018-03-29 19:28:14
tags: js
---

函数是一种特殊的对象，在js中运用很广泛。函数的定义是一段可以反复调用的代码块。

## 函数声明的几种方式

函数声明跟变量声明比较类似，同时也存在声明提升。

```javascript
//基本函数声明
function f1(x,y){
    return x+y
}//会提升

//函数表达式
var f2 = function(x,y){
    return x+y
}//匿名函数赋值给一个变量

//new方法
var f3 = new Function('x','y','return x+y')

//ES6箭头函数
var f4 = (x,y)=>{return x+y}
```

因为函数是对象，所以函数名相当于函数的地址引用，函数体存在堆内存中，通过地址引用访问。

## this

### this指向
初学js的时候，很难搞清楚this的指向，原来是js做了很鸡贼的处理，所以新手接触的时候往往摸不着头脑。在开头，先说结论：

**this 实际上是在函数被调用时进行绑定，它的指向完全取决与函数在哪里被调用，也就是说在函数被调用之前，this是无法被确定的。**

### 绑定规则

#### 默认绑定

这类绑定是最常用的调用类型，独立函数调用。

```javascript
    function foo(){
        var a = 1
        console.log(this.a)
    }
    var a = 2
    foo() //2
```

在非严格模式下，this指向了全局对象，所以a就是全局变量a。因为它是独立调用函数，没有任何的修饰。

如果在严格模式下，this会指向undefined。

```javascript
    function foo(){
        'use strict'
        var a = 1
        console.log(this.a)
    }
    var a = 2
    foo() //Uncaught TypeError: Cannot read property 'a' of undefined
```

此时this指向了undefined，所以会报错。

#### 隐式绑定

如果一个函数被某个对象拥有或者包含，在调用时即为隐式调用，this会指向调用人。

```javascript
    function foo(){
        var a = 1
        console.log(this.a)
    }
    var obj = {
        a: 2,
        foo: foo
    }
    foo() //undefined
    obj.foo() //2
```

foo()是对调用函数，指向的是全局对象，因为全局对象a没有被赋值，所以结果为undefined。

obj.foo存储了foo函数的地址，调用时为隐式调用，this指向的就是obj这个对象，所以this.a也就是obj对象里面的a。

```javascript
    function foo(){
        var a = 1
        console.log(this.a)
    }
    
    var obj2 = {
        a: 3,
        foo: foo
    }
    
    var obj1 = {
        a: 2,
        obj2: obj2
    }

    foo() //undefined
    obj1.obj2.foo() //3
```

如果有多层关系，则最后一层会在调用位置上起作用，即指向的是obj1.obj2。

#### 显示绑定

函数自带有call()及apply()方法，可以利用这两个方法来显示指定this绑定，绑定值为第一个参数。

```javascript
    var a = 3
    
    function foo(){
        var a = 1
        console.log(this.a)
    }
    
    var obj = {
        a: 2
    }
    
    foo.call(obj) //2
    foo.call(undefined) //3
    foo.call() //3
```

只要使用了call()方法，第一参数传什么，那么this就是什么，很容易理解。如果什么都不传，那么第一个参数会默认为undefined，在非严格模式下，this指向undefined会变成指向全局对象，如果是严格模式下，那么传什么就指向什么。

```javascript
    function foo(){
        'use strict'
        console.log(this)
    }
    
    function bar(){
        console.log(this)
    }
    
    foo.call() //undefined
    bar.call() //Window对象

```

#### new绑定

构造函数，用new来调用，则指向的是构造出的新对象。

```javascript
    function foo(a){
        this.a = 2
    }
    
    var bar = new foo(2)
    console.log(bar.a) //2
```

### this绑定总结

1. 函数为new绑定，则this指向的是新创建的对象。
2. 如果通过call()、apply()进行显示绑定的，则this指向方法第一个传入的参数。
3. 如果为在对象中调用，则为隐式绑定，this指向的是最后一层的调用对象。
4. 如果是独立调用函数，则this指向全局对象。

