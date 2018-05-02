---
title: 前端基础知识整理系列（7）：JSONP
date: 2018-04-28 23:07:13
tags: js
---

浏览器为了安全的目的，制定了同源政策。这是为了保证用户信息安全，防止恶意的网站窃取数据。([阮一峰 浏览器同源政策及其规避方法](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html))

但实际工作中，我们避免不了在不同源的客户端跟服务端进行信息交互。所以前辈们想出了许多办法去规避同源政策，今天我先介绍第一种跨域方法-JSONP。

## 历史
其实在2005年AJAX出现之前，已经有程序员发明出了JSONP，目的就是为了优化用户体验。在JSONP发明之前，还有更多的可代替方案，虽然随着技术迭代已经消声灭迹，但我觉得还是有必要提及下，这能更好的理解新技术发明解决的痛点。

### form表单

我们知道form表单有action属性，可以触发http请求。最早的时候，前后端数据交互都是基于这个形式。

前端想向后端请求一个数据，那么写一个form表单，然后提交触发，后端收到请求后返回响应。

但是form表单在提交的时候会刷新页面，用户体验不好，注重体验的那批程序员想了一种方法，那就是通过html标签元素触发http请求。

### img标签发送请求

浏览器在碰到有img标签的时候，会自动向服务器发起请求，所以这就是可以利用的。
```HTML
    <div>
        <span>余额：</span>
        <span id="balance">100</span>
    </div>
    <button id="paymentBtn">付款</button>
```
```javascript
/*javascript代码*/
paymentBtn.onclick = function(){
    var img = document.createElement('img')
    img.src = '/pay'

    img.onload = function(){
        alert('success')
        balance.innerText = parseInt(balance.innerText-1,10) 
        //响应成功则余额减1
    }

    img.onerror = function(){
        alert('fail')
    }
}

```

只要在script标签内创建一个img标签，在此标签加载的时候会发起请求，后端收到请求，返回响应结果。如果状态码返回2xx，那么说明这次请求成功，前端便会执行img.onload的代码。

虽然达到了通信效果，但这样后端必须传回一个真实的img文件浏览器才会认为响应成功。

还有其他方式吗？

### script标签

可以发送请求的标签，除了img，还有a、link、script等等。a标签因为需要点击，所以排除。那剩下的选择也不多了，就想到了用script来代替。

到了这一步，其实离JSONP已经很接近了。


```javascript
/*javascript代码*/
window.xxx = function(data){
    if(data==='success'){
        alert('这是前端代码，数据是'+data)
        balance.innerText = parseInt(balance.innerText-1,10) 
    }
}
paymentBtn.onclick = function(){
    var script = document.createElement('script')
    script.src = '/pay'
    docuent.body.appendChild(script)
    //script必须在页面中才能发起请求
    
    script.onload = function(e){
        alert('success')
        e.currentTarget.remove()  //清除script标签
    }

    img.onerror = function(e){
        alert('fail')
        e.currentTarget.remove()  //清除script标签
    }
}
```

这样的方法跟img很类似，但是通过创建script标签可以做更多的事情。

先看下后端代码

```javascript
//node.js
if (path === '/pay') {
        var balance = fs.readFileSync('./db', 'utf8')
        //db文件只存了数值100，为了模拟数据库的作用
        var newBalance = balance - 1
        fs.writeFileSync('./db', newBalance)
        
        response.setHeader('Content-Type', 'application/javascript')
        response.statusCode = 200
        response.write(`
            xxx.call(undefined,'success')
        `)//向前端传的date是'success'
    
        response.end()
    ｝

```

但这个有点不好的是，前后端会耦合在一起，因为后端程序员必须去知道前端传过来函数的名字xxx，才能相应的返回数据。

为了解耦，因此约定，统一传过来的函数名为callback，后端通过查询参数获得，这就是JSONP。

## JSONP（JSON + padding）

因为script标签的请求不受域名限制，所以不同源之间的通信可以利用这一点。

**JSONP定义：通过动态创建script元素，插入到页面中，在加载时向服务器请求数据；服务器收到请求后，通过指定名字的回调函数将数据传回来。**

前端代码

```javascript
paymentBtn.addEventListener('click',function(){
    
    var script = document.createElement('script')

    var functionName = 'banlance'+ parseInt(Math.random()*100000,10)
    //定义随机的回调函数名
    
    window[functionName] = function(result){
        if(result === 'success'){   
            // alert('这是前端写的代码')
            balance.innerText = parseInt(balance.innerText-1,10) 
        }else{
            alert('页面出错')
        }
    }
    
    script.src = 'http://127.0.0.1:8081/pay?callback='+functionName
    //本身的域名端口为 http://127.0.0.1:80，因为端口号不同，所以不是统一个源
    document.body.appendChild(script)

    script.onload = function(e){

        e.currentTarget.remove()
        delete window[functionName]
        //执行完毕删除回调函数
    }

    script.onerror = function(e){
        alert('fail')
        e.currentTarget.remove()
        delete window[functionName]
    }
})
```

后端代码

```javascript
if (path === '/pay') {
    var balance = fs.readFileSync('./db', 'utf8')
    var newBalance = balance - 1
    fs.writeFileSync('./db', newBalance)
    response.setHeader('Content-Type', 'application/javascript')
    response.statusCode = 200
    response.write(`
        ${query.callback}.call(undefined,'success')
    `)

    response.end()
}
```

这样不管我前端写的函数名是什么，后端通过查询callback就能得到，从而达到通信的效果。

**为什么叫做JSONP呢，其实后端返回的格式不一定是JSON格式的字符串，也可以是其他的形式。只是工作中，JSON作为一门数据交换语言已经被广泛应用，一般来说后端返回的是JSON格式的字符串方便前端处理，用的多了，也就叫做JSONP了，严格来说跟JSON没多大关系。**

### jQuery实现

jQuery写法很简单，因为其帮我们封装了一系列的代码，节省了步骤。

```javascript
$(paymentBtn).on('click', function () {
    $.ajax({
        url: "http://127.0.0.1:8081/pay",
        dataType: "jsonp",
        success: function (data) {
            if (result === 'success') {
                // alert('这是前端写的代码')
                balance.innerText = parseInt(balance.innerText - 1, 10)
            }
        },
        error: function () {
            alert('fail');
        }
    });
```
不用写查询参数，也不用在script加载完后删除，jQuery都帮我们做好了~~

*ps：JSONP只能用get方法做请求，因为创建script没有提供其他的api，默认就是 get方法*

## 感受

前端的发展能到今天，真的是离不开各位前辈们苦思冥想的铺垫。如果不是有追求极致的人，那么或许今天的交互还停留在表单提交的状态。

所以一直抱着敬畏之心去学习，虽然陈旧的技术被淘汰，但我还是愿意去了解它们，因为这些是当时前辈们的思想火花，对前端发展有重要的参考意义。