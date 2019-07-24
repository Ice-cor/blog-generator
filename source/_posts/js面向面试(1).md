---
title: js面向面试(1)
date: 2019-07-24 18:05:21
tags:
---

## 同步阻塞

```javascript
function execTime(){
    //补全代码
}

console.log(1)
execTime(3000)
console.log(2) //3秒后打印出2
```

思路：因为异步代码执行是不在主线程上的，无法阻塞同步代码的执行，所以得增加同步代码的执行时间。

方法：

```javascript
function execTime(t){
    var start = Date.now()
    
    while(Date.now() - start < t){
        //小于3000ms，在栈内，否则出栈
    }
}
console.log(1)
execTime(3000)
console.log(2) //success
```

如果不局限与这个形式，其实可以用async/await来解决，即把异步变成同步。

```javascript
function waitTime(t) {
    return new Promise(resolve => {
        setTimeout(_ => {
          resolve()
        }, t)
    })
}

(async ()=>{
     console.log(1)
     let b = await waitTime(3000)
     console.log(2)
})() //3秒后打印出2
```

## 防抖节流函数

#### 节流函数

先说个实际场景，判断滚动条是否到底部，然后加载更多。

```javascript
    document.addEventListener('scroll', ()=>{
        console.log('scroll')
    })
```

这样滚动的时候，在监听下，会执行多次代码逻辑，比较影响性能。

![多次执行代码](https://zwqblog.oss-cn-shenzhen.aliyuncs.com/js%E9%9D%A2%E8%AF%95/scrollListen.gif)

节流函数的存在作用就在于可以限定一段时间才去执行一次代码。

```javascript
    let lock = false
    document.addEventListener('scroll', () => {
      if (lock) return
      lock = true
      setTimeout(() => {
        lock = false
        console.log('scroll')
      }, 200)
    })
```

原理也很简单，就是设定一个锁，然后异步解开便可。

![节流函数限制](https://zwqblog.oss-cn-shenzhen.aliyuncs.com/js%E9%9D%A2%E8%AF%95/scrollListen2.gif)

封装一下：

```javascript
    function scrollBottom(){
        if(scroll === bottom){
            loadMore()
        }
    }
    function throttle(fn, time){
        let lock = false
        return ()=>{
            if(lock) return
            lock = true
            setTimeout(()=>{
                lock = false
                fn()
            },time)
        }
    }
    document.addEventListener('scroll',throttle(scrollBottom,200))
```

#### 防抖函数

有这么个应用场景，为了更好的用户体验，input框输入用户名的时候，就向后端发起请求，用于检测此用户名是否注册过。

```javascript
     input.addEventListener('input',function(){
          ajax(...)
      })
```



![监听input](https://zwqblog.oss-cn-shenzhen.aliyuncs.com/js%E9%9D%A2%E8%AF%95/inputListen.gif)

这样，在用户进行输入操作的时候，会频繁的向后端发送请求，增大了服务器压力。

防抖函数的作用，在于在一定时间后，如果没有新操作，才会去调用函数。

```javascript
    function ajax() {
      send(...)
    }
    function debounce(fn, time) {
      let timeId = null
      return () => {
        clearTimeout(timeId)
        timeId = setTimeout(fn, time)
      }
    }
    input.addEventListener('input', debounce(ajax, 200))

```

![防抖函数](https://zwqblog.oss-cn-shenzhen.aliyuncs.com/js%E9%9D%A2%E8%AF%95/inputListen2.gif)


这样，可以大大减少向后端发请求的次数，并且体验上也非常好。