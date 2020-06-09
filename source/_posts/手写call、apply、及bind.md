---
title: 手写call、apply、及bind
date: 2020-06-04 23:09:15
tags: js bind
---

call、apply、及bind有相同的作用，就是可以手动绑定this的指向，并且call、apply调用后会立即执行，且可以穿参，bind则是不立即执行；

## call

先看看call的用法：

```javascript
function fn(arg1, arg2){
	console.log(this.name)
  console.log(arg1 + arg2)
}
var name = "张三";
let obj = {name: "李四"};

fn(1, 2) // 打印出 “张三” 和 3
fn.call(obj, 1, 2) // 打印出 "李四" 和 3
```

很显然，函数如果是隐式调用，this则为undefined，在非严格模式下，会默认指向Window对象；而使用call调用后，this会指向传入的obj对象；
<br />实现：

```javascript
Function.prototype.myCall = function(context, ...args) {
  if (typeof context === 'undefined' || context === null) {
    context = window
  } else {
    // 将非对象参数包装成对象；
    context = Object(context);
  }
  // 防止覆盖掉原有属性
  const key = Symbol();
  // 根据隐式调用的规则，此处的this，指向调用myCall的函数
  context[key] = this;
  // 方法执行
  const result = context[key](...args);
  delete context[key];
  return result;
};

var name = "张三";
let obj = {name: "李四"};

function fn(arg1, arg2){
  // context[key](...args) 这里的执行，绑定了此处的this为context，即传入的对象
	console.log(this.name)
  console.log(arg1 + arg2)
};

fn.myCall(obj, 1, 2); // 打印出 "李四" 和 3
```

## apply
apply与call的功能差不多，只是apply可以传入参数的数组；<br />
示例：

```javascript
function fn(arg1, arg2){
	console.log(this.name)
  console.log(arg1 + arg2)
}
var name = "张三";
let obj = {name: "李四"};

fn(1, 2) // 打印出 “张三” 和 3
fn.apply(obj, [1, 2]) // 打印出 "李四" 和 3
```

实现：

```javascript
Function.prototype.myCall = function(context, args) {
  if (typeof context === 'undefined' || context === null) {
    context = window
  } else {
    // 将非对象参数包装成对象；
    context = Object(context);
  }
  // 防止覆盖掉原有属性
  const key = Symbol();
  // 根据隐式调用的规则，此处的this，指向调用myCall的函数
  context[key] = this;
  // 方法执行，用展开符展开数组
  const result = context[key](...args);
  delete context[key];
  return result;
};

var name = "张三";
let obj = {name: "李四"};

function fn(arg1, arg2){
  // context[key](...args) 这里的执行，绑定了此处的this为context，即传入的对象
	console.log(this.name)
  console.log(arg1 + arg2)
};

fn.myCall(obj, 1, 2); // 打印出 "李四" 和 3
```

## bind
bind的作用，可以手动绑定this指向，并且可以传参，此次不调用函数；<br />
<br />示例：

```javascript
function fn(arg1, arg2) {
  console.log(this.name);
  console.log(arg1 + arg2);
}
var name = "张三";
let obj = { name: "李四" };
let foo = fn.bind(obj, 1, 2);
foo(); // 打印出 “李四” 和 3
```
实现：<br />

```javascript
Function.prototype.myBind = function(context, ...args) {
  if (typeof context === 'undefined' || context === null) {
    context = window;
  } else {
    context = Object(context);
  }
  const _this = this
  return function(...args2) {
    return _this.call(context, ...args, ...args2);
  };
};

function fn(arg1, arg2) {
  console.log(this.name);
  console.log(arg1 + arg2);
}

var name = "张三";

let obj = { name: "李四" };
let foo = fn.myBind(obj, 1, 2);
let foo2 = fn.myBind(obj);
let foo3 = fn.myBind(undefined)

foo(); // 打印 "李四" 和 3
foo(4, 5) //  动态传参 优先打印 "李四" 和 3
foo2(2, 3) // 动态传参 打印 "李四" 和 5
foo3(5, 6) // 动态传参 打印 "张三" 和 11 (var name会挂载到全局window对象上)
```

<br />这样是最简单的实现，能实现基本的绑定this指向及传参功能。<br />

### 不足
不支持new操作。<br />
<br />具体为什么不支持，下篇手写new的时候一并解释。<br />

