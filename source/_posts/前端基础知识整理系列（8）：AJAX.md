---
title: 前端基础知识整理系列（8）：AJAX
date: 2018-05-03 03:05:55
tags: JS AJAX
---
AJAX全名为 Asynchronous Javascript and XML，中文译名异步的Javascript和XML。
其实在JSON语言被发明出来之前，前后端数据交互用的是XML，随着JSON被发明，XML逐渐被这一更简单易用的语言所取代。

这次记录下怎么写一个常规的AJAX请求。

## HTTP请求头

用AJAX，就是为了发一个HTTP请求，所以必须对HTTP有所了解。先简单介绍下HTTP报文所代表的含义.

[附上链接](https://blog.csdn.net/u010256388/article/details/68491509)


## AJAX基本写法

对HTTP有了大概的了解后，学习AJAX就简单多了~

```javascript
let request = new XMLHttpRequest() 
//XMLHttpRequest()是客户端提供用于数据传输的API

request.open(method,url) 
//method为请求方法
//url是请求地址或者路径

request.onreadystatechange = function(){
    //onreadystatechange监听请求状态的改变，readyState为4表示请求完成，但不能判断请求是否成功，只能说明请求完成了
    
    if (request.readyState === 4) {
            if (request.status === 200 ) {
            //如果请求完毕，且状态码返回2xx，表示请求成功
                let string = request.responseText
                let object = JSON.parse(string)
                success.call(undefined, object)
                //回调函数，在请求成功时调用，参数传入后端返回的JSON解析数据
            } else if (request.status > 400) {
            //如果请求完毕，且状态码返回4xx，表示请求失败
                fail.call(undefined, request)
            }
    }
}

request.send(body)
//设置请求体，向服务端发送

```

这就是AJAX基本写法。

## 封装一个AJAX函数

自己封装一个AJAX函数，方便以后工作使用，以免用的时候才去查API。

```javascript
window.zwq = {}
//命名空间

zwq.ajax = function(method,url,body,success,fail){
    let request = new XMLHttpRequest() 

    request.open(method,url) 

    request.onreadystatechange = function(){
        if (request.readyState === 4) {
                if (request.status === 200 ) {
                    let string = request.responseText
                    let object = JSON.parse(string)
                    success.call(undefined, object)
                } else if (request.status > 400) {
                    fail.call(undefined, request)
                }
        }
}

request.send(body)

}

/*下面是函数调用*/

zwq.ajax(
'GET',
'/xxx',
'你好呀',
function(text){console.log(text)},
function(request){console.log(request)}
)
```
可以看出，常规的封装，在调用的时候很麻烦，而且信息不明确，不去看源代码根本不知道每个参数所代表的意思。

优化一下

```javascript
window.zwq = {}
//命名空间

zwq.ajax = function({method,url,body,success,fail}){
    //es6解构赋值
    let request = new XMLHttpRequest() 

    request.open(method,url) 

    request.onreadystatechange = function(){
        if (request.readyState === 4) {
                if (request.status === 200 ) {
                    let string = request.responseText
                    let object = JSON.parse(string)
                    success.call(undefined, object)
                } else if (request.status > 400) {
                    fail.call(undefined, request)
                }
        }
}

request.send(body)

}

/*下面是函数调用*/

zwq.ajax({
    method:'GET',
    url:'/xxx',
    body:'你好呀',
    success:function(text){console.log(text)},
    fail:function(request){console.log(request)}
})
```

这样，在调用的时候传入一个对象，把数据整合起来，看起来就很统一很明确了。

## 借鉴jQuery的AJAX封装

先看一下jQuery的AJAX调用方式。

```javascript
jQuery.ajax({
    method:'GET',
    url:'/xxx',
    body:'你好呀'
}).then(
    function(text){console.log(text)},
    function(request){console.log(request)}
)
```

把回调函数分离出来，便于书写管理，这个到底怎么实现呢？

其实用Promise就行了

```javascript
window.zwq = {}
//命名空间

zwq.ajax = function({method,url,body,success,fail}){
    //es6解构赋值
    return new Promise(function(resolve,reject){
    //约定Promise对象所传入函数的参数为resolve跟reject
    //resolve为成功所执行的函数，reject为失败所执行的函数
        let request = new XMLHttpRequest() 

        request.open(method,url) 
    
        request.onreadystatechange = function(){
            if (request.readyState === 4) {
                    if (request.status === 200 ) {
                        let string = request.responseText
                        let object = JSON.parse(string)
                        resolve.call(undefined, object)
                    } else if (request.status > 400) {
                        reject.call(undefined, request)
                    }
            }
        }

        request.send(body)
    })
}

/*下面是函数调用*/

zwq.ajax({
    method:'GET',
    url:'/xxx',
    body:'你好呀'
}).then(
    (text)=>{console.log(text)},
    (request)=>{console.log(request)}
)
//Promise对象返回附带有then的方法，所以可以这样调用
```

这样就能简单模拟jQuery的AJAX方法发请求了~~~