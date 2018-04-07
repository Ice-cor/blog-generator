---
title: 原生js实现jQuery的api
date: 2018-04-07 10:35:07
tags: js
---

写的原生js越多，越能体会到jQuery的好。

在实现需求的时候，如果不引入jQuery，得费很大功夫去查找dom，因为原生js的api实在是很烂，语法不简洁不说，这些方法还有很多不一致性，如果不仔细阅读文档，会产生很多迷糊。

趁着空闲，封装了几个类似jQuery的api，在此记录下实现的细节。

## 先实现方法

因为经验不足，如果一开始就想着优化代码是很难写下去的，所以先逐个把功能实现，再来简化。

这次要实现像jQuery里面传入选择器就能查找到dom节点、添加class、设置文本的功能。

HTML部分：

```HTML
<ul>
    <li id="item1">选项1</li>
    <li id="item2">选项2</li>
    <li id="item3">选项3</li>
    <li id="item4">选项4</li>
    <li id="item5">选项5</li>
</ul>
```
js部分：

```javascript

function findDomNode(selector){
    let nodes = document.querySelectorAll(selector)
    return nodes
}
//利用元素的querySelectorAll方法查找选择器对应的节点，返回的是一个伪数组。

function addClass(node,classes){
    classes.forEach((value)=>{
        for(let i = 0; i<node.length; i++){
            node[i].classList.add(value)
        }
    })
}

//通过传入数组跟节点，然后遍历数组跟节点，把数组的每一项值给到每一个节点上


function text(node,text){
    for(let i = 0; i<node.length; i++){
            node[i].textContent = text
        }
}

let li = findDomNode('ul > li')
addClass(li,['a','b','c'])
text(li,'你好吗')
```

这样就简单的实现了功能，但是这样的代码并不优雅，因为每一个方法都是独立的，没有关联性。

## 优化一 - 命名空间

```javascript
let myMethod = {}

myMethod.findDomNode = function findDomNode(selector){
    let nodes = document.querySelectorAll(selector)
    return nodes
}

myMethod.addClass= function(node,classes){
    classes.forEach((value)=>{
        for(let i = 0; i<node.length; i++){
            node[i].classList.add(value)
        }
    })
}

myMethod.text = function(node,text){
    for(let i = 0; i<node.length; i++){
            node[i].textContent = text
        }
}


let li = myMethod.findDomNode('ul > li')
myMethod.addClass(li,['a','b','c'])
myMethod.text(li,'你好吗')
```

这样用一个对象，把所有方法囊括其中，看起来会有一些关联，重点是如果在团队合作中，不会干扰到其他人的代码，因为如果写到全局对象中有可能方法名跟变量名会重复。

## 优化二 - 链式操作

jQuery的链式操作写起来让人大呼过瘾，上面的代码需要在每个方法传入node节点，还是很繁琐的，改写一下，让其可以链式操作。

### 方法一

```javascript

window.myMethod = function(selector){
    let nodes = {}
    let temp = document.querySelectorAll(selecter)
    for (let i = 0; i < temp.length; i++) {
        nodes[i] = temp[i]
    }
    nodes.length = temp.length
    /*
    以上声明一个nodes对象，让其变为一个伪数组，存储了node节点；
    使用伪数组是因为节点有可能是一个或者多个；
    然后本身因为是对象，方便添加方法；
    利用temp临时变量存储获取到的节点，然后赋值给nodes，最终nodes是纯净的一个伪数组，原型链直接指向Object，当然不做这一步也是可以的；
    */
    
    nodes.addClass = function () {
        for (let key in arguments) {
            for (let i = 0; i < nodes.length; i++) {
                nodes[i].classList.add(arguments[key])
            }
        }
    }
    //利用arguments拿到传入的参数，因为arguments也是一个伪数组，用for...in拿到arguments每项的value，然后用原生dom方法给需要的节点添加class

    nodes.text = function (text) {
        if (text === undefined) {
            let texts = []
            for (let i = 0; i < nodes.length; i++) {
                texts.push(nodes[i].textContent)
            }
            return texts
        } else {
            for (let i = 0; i < nodes.length; i++) {
                nodes[i].textContent = text
            }
        }
    }
    //这里仿照了下jQuery的功能，及不出入参数的时候，默认就返回节点的文本，传入了参数就改变文本

    return nodes
}


let li = myMethod('li')
li.addClass('a', 'b', 'c')
li.text('你好呀')
```

### 方法二

```javascript

window.myMethod = function(selector){
    let nodes = {}
    let temp = document.querySelectorAll(selecter)
    for (let i = 0; i < temp.length; i++) {
        nodes[i] = temp[i]
    }
    nodes.length = temp.length
    
    return {
        element: nodes,
        addClass: function () {
            for (let key in arguments) {
                for (let i = 0; i < nodes.length; i++) {
                    nodes[i].classList.add(arguments[key])
                }
            }
        },
        text: function (text) {
            if (text === undefined) {
                let texts = []
                for (let i = 0; i < nodes.length; i++) {
                    texts.push(nodes[i].textContent)
                }
                return texts
            } else {
                for (let i = 0; i < nodes.length; i++) {
                    nodes[i].textContent = text
                }
            }
        }
    }
}

let li = myMethod('li')
li.addClass('a', 'b', 'c')
li.text('你好呀')
```

方法一跟方法二大相径庭，只是说方法一返回的是节点对象，然后这个对象包含有一系列的方法；而方法二则是返回节点对象及一系列操作方法，看个人喜好吧。


## 改个名字

```javascript

window.jQuery = function(selector){
    let nodes = {}
    let temp = document.querySelectorAll(selecter)
    for (let i = 0; i < temp.length; i++) {
        nodes[i] = temp[i]
    }
    nodes.length = temp.length
    
    nodes.addClass = function () {
        for (let key in arguments) {
            for (let i = 0; i < nodes.length; i++) {
                nodes[i].classList.add(arguments[key])
            }
        }
    }

    nodes.text = function (text) {
        if (text === undefined) {
            let texts = []
            for (let i = 0; i < nodes.length; i++) {
                texts.push(nodes[i].textContent)
            }
            return texts
        } else {
            for (let i = 0; i < nodes.length; i++) {
                nodes[i].textContent = text
            }
        }
    }

    return nodes
}

window.$ = jQuery

let li = $('li')
li.addClass('a', 'b', 'c')
li.text('你好呀')
```

这样就能用$()这个方法去查找节点了~

并且返回的nodes对象带有方法，可以调用~