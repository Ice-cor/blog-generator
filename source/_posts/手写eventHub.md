---
title: 手写eventHub
date: 2020-05-21 02:17:19
tags: js eventHub
---

eventHub是前端常用到的技巧之一，不同模块之间的通信，除了使用storage，还能自己写一个eventHub完成需求；<br />
<br />eventHub是采用发布/订阅模式进行设计的，直接上代码：<br />

<a name="jpkEu"></a>
## 代码
```javascript
// eventhub.js

class CreateEventHub {
  private cache = {}; // 存放数据
  emit(eventName, fn) { // 触发函数
    this.cache[eventName] = this.cache[name] || [];
    this.cache[eventName].push(fn); // 将需要执行的函数推进数组
  }
  on(eventName, data) { // 监听函数
    // data为回调传过来的数据
    (this.cache[eventName] || []).forEach(fn => {
      fn(data); // 以此执行对应数组内的函数
    });
  }
  off(eventName, fn) { // 停止执行函数
    this.cache[eventName] = this.cache[name] || [];
    const index = this.cache[eventName].indexOf(fn);
    if(index !== -1){
      this.cache[eventName].splice(index, 1); // 删除对应的函数
    }
  }
}
const eventHub = new CreateEventHub(); 
export default eventHub;
export CreateEventHub;
```


```javascript
// 1.js
import eventHub from "./eventHub";

eventHub.emit("a", data => {
  console.log("a");
  console.log(data);
});

```


```javascript
// 2.js
import eventHub from "./eventHub";

setTimeout(() => {
  eventHub.on("a", "回调"); // 2秒后打印出'a' 及传过去的数据'回调'
}, 2000);

```


```javascript
// 3.js
import eventHub from "./eventHub";

const fn = data => {
  console.log("b");
  console.log(data);
};

eventHub.emit("b", fn);
eventHub.off("b", fn);
eventHub.on("b", "回调b"); // 此处不执行，因为off方法已经将eventHub内b数组内的函数清掉了

```
<a name="gCNNv"></a>
## 结尾
大致的思路就是这样，具体的问题可以根据需求做调整，例如回调多个数据或者说清除整个对应的eventName内的数据等等，只需要稍微调整参数类型及对应代码即可；
