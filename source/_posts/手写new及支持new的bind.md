---
title: 手写new及支持new的bind
date: 2020-06-08 22:10:34
tags: js new
---

## new的命令与原理

<br />**new运算符**创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。new 关键字会进行如下的操作：

1. 创建一个空的简单JavaScript对象（即{}）；
1. 链接该对象（即设置该对象的构造函数）到另一个对象 ；
1. 将步骤1新创建的对象作为this的上下文 ；
1. 如果该函数没有返回对象，则返回this。
> 摘自 MDN new篇章

### 效果：

```javascript
// new 操作符
function Person(name, age) {
  this.name = name;
  this.age = age;
}

let o = new Person('张三', 18)

o.name === '张三' // ture
o.age === 18 // true
o.constructor === Person // true

// 如果构造函数有返回值

function Person1(name, age) {
  this.name = name;
  this.age = age;
  return {name: '李四'};
}

function Person2(name, age) {
  this.name = name;
  this.age = age;
  return 'hi';
}

let o1 = new Person1('张三', 19);
let o2 = new Person2('王五', 20)

o1.name // '李四' 构造函数返回值是对象，则为此对象
o2.name // ‘王五’ 构造函数返回值是基础类型，则返回this，此时的this指向的是new创建的对象
```

### 原理解析
在使用**new运算符**的时候，会经历以下过程：

```javascript
// new 操作符
function Person(name, age) {
  this.name = name;
  this.age = age;
}

let o = new Person('张三', 18)

//  模拟过程
function Person(name, age) {
  let obj = {}; // 创建一个新对象
  obj.__proto__ = Person.prototype // 将构造函数的原型复制到新对象上
  
  this.name = name;
  this.age = age;
  
  // 没有写返回值则默认返回this
  // this在new执行时，会指向新创建的对象obj
  return this
}
```

### 关键
从代码来看，手写new的关键在于：

1. 通过构造函数返回值来判断返回什么东西，如果没有return(默认返回undefined)则会返回this;
1. this的指向为新创建的对象；
1. 实例对象的constructor会是这个构造函数;


## 手写new

```javascript
// 手写new

/** 
  *@param(Function) fn 为传入的构造函数
  *@param(any) args 为构造函数的所有参数
  */
function myNew(fn, ...args) {
  let obj = {}; // 创建一个对象
	
  // __proto__不是标准属性，项目中不建议使用
  // 将构造函数的原型复制到新对象上
  obj.__proto__ = fn.prototype;

  let result = fn.call(obj, ...args); // 绑定函数的this指向obj，并执行
  
  // 如果构造函数有返回值切为对象，返回这个构造函数的返回值
  // 否则返回新创建的对象obj
  return result instanceof Object && result !== null ? result : obj;
}

function Person(name, age) {
  this.name = name;
  this.age = age;
}

function Person2(name, age) {
  this.name = name;
  this.age = age;
  return {name: '李四'}
}

Person.prototype.sayHi = function() {
  console.log("hi");
};
Person2.prototype.sayHi = function() {
  console.log("hi");
};

// 测试

let newObj = myNew(Person, "张三", 18);
newObj.name // 张三
newObj.sayHi() // 'hi'
newObj.constructor === Person // true

let newObj2 = myNew(Person2, "张三", 18);
newObj2.name // 李四
newObj.constructor === Person // false

```

<br />其实只要明白了执行原理，手写new也不是难事。


## 支持new的bind

<br />先看看上次写的bind代码：

```javascript
Function.prototype.myBind = function(context, ...args) {
  if (typeof context === 'undefined' || context === null) {
    context = window;
  } else {
    context = Object(context);
  }
  const _this = this
  return function bindResult(...args2) {
    return _this.call(context, ...args, ...args2);
  };
};
```

### 存在问题
在上面new的解析时提到，**如果该函数没有返回对象，则返回this，this指向新创建的对象；**<br />**

```javascript
function Person(name, age) {
  this.name = name
  this.age = age
}
let obj = {name: '张三', age: 18}
let foo = Person.myBind(obj) // 此时foo的值为function bindResult，this指向obj

// 函数 bindResult 的返回值为 _this.call(context, ...args, ...args2)
// 此处 _this 指向的是传入的构造函数 Person, Person 执行时并没有返回值
// 所以执行new时，会返回新创建的对象，因为没有绑定属性所以是 {}
// 而且还会改变原有obj的值
let newFoo = new foo() // {}
/** 相当于执行
	function bindResult(...arg2) {
  	let obj = {}
    obj.__proto__ = bindResult.prototype
    return _this.call(context, ...args, ...args2); // return undeined;
    // 等同于 return this; this指向obj即 {}
  }
*/

obj // {name: undefined, age: undefined} (Person.call(obj, undefined, undefined))

// 正常使用bind
let person1 = Person.bind(obj, '李四', 20)
let person2 = new person1() // person2 为对象 {name: '李四', age: 20}
obj // {name: '张三', age: 18}
```

<br />与正常的 **bind **方法对比，很明显可以看出 new 执行时，this仍然没有改变，为原来的 obj ，这就跟原版执行 new 的情况不一样，所以需要改进。<br />

### 改进

需要改进的方面主要时检测是否使用了 new运算符， 然后针对此类情况做相对于的操作。<br />

```javascript
Function.prototype.myBind = function(context, ...args) {
  if (typeof context === 'undefined' || context === null) {
    context = window;
  } else {
    context = Object(context);
  }
  const _this = this
  const bindResult = function(...agrs2) {
      return _this.call(
      // 在new运算符中，this指向新创建的对象
      // 判断此对象的构造函数是否为bindResult
      // 如果为真，则使用了new运算符，返回this即可
      this instanceof bindResult ? this : context, 
      ...args, 
      ...args2
    );
  }
	if(this.prototype){
    bindResult.prototype = this.prototype // 将bindResult的原型指向构造函数的原型
  }
  return bindResult;
};

// 测试
function Person(name, age) {
  this.name = name
  this.age = age
}
let obj = {name: '张三', age: 18}
let foo = Person.myBind(obj) // 此时foo的值为function bindResult，this指向obj
let newFoo = new foo('李四', 20) // {name: '李四', age: 20}
obj // {name: '张三', age: 18}
```

### 总结
重点在于：

1. 通过call或者apply强绑定this指向；
1. 可以动态传参及静态传参；
1. 检测是否使用new运算符，从而返回对应的值；
