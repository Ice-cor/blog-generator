---
title: 巧遇BUG-理解事件委托、冒泡、捕获
date: 2018-03-19 04:09:04
tags: bug
---
今天做一个需求，遇到一个bug，苦思冥想后，算是解决了，现在记录一下。
## 烦人的bug
html代码：
```html
<strle>
    ul {
        list-style: none;
        width: 150px;
        height: 400px;
        background: yellow;
    }

    span {
        width: 50px;
        height: 50px;
        background: red;
        display: block;
        border: 1px solid black;
        margin-bottom: 50px;
    }

    li {
        border: 1px solid black;
        width: 100px;
        background: green;
    }
</style>

<!--下面是html-->
<ul id="grandpa">
        <li id="father1">
            <span id="children1"></span>
            <span id="children2"></span>
            <span id="children3"></span>
        </li>
    </ul>
```

<center><img src="http://ww3.sinaimg.cn/large/0060lm7Tly1fphmy6nr6lj30640byglg.jpg" alt="图1"></center>
这是效果，我要点击每个红色的span触发相对应的事件
传统的事件必须每个span加一个click事件，事件委托则可以减少click操作，把事件委托在父级上，父级通过事件捕获，传递到子元素（但如果对孙子级别则委托不了的，需要两层的click）

```javascript
    var grandpa = document.getElementById('grandpa')
    var father1 = document.getElementById('father1')

    grandpa.onclick = function (event) {
        alert(1)
        father1.onclick = function (e) {
            var target = e.target  //找到目标元素
            console.log(target)
            alert(2)
           target.style.background = "yellow"
        }
    }
```
这样，我点击对应的红色span的时候背景会变黄色，但有个问题需求是相对应点击的变黄，其他的span仍然维持红色，所以做了个类似jQuery的siblings方法，即所选中目标的其他同胞元素

改进一下

```javascript
    var grandpa = document.getElementById('grandpa')
    var father1 = document.getElementById('father1')

    grandpa.onclick = function (event) {
        alert(1)
        
        father1.onclick = function (e) {
            var target = e.target
            console.log(target)
            alert(2)
            for (let i = 0; i < this.children.length; i++) {
                /*
                *this为li元素，this.children能找到下面的所有子元素
                *target为所点击的目标元素
                * 循环让target跟每个子元素做比对，是当前那个则改变，不是的则不变
                * */
                if (target !== this.children[i]){
                    this.children[i].style.background = "red"
                    
                }else{
                    target.style.background = "yellow"
                    
                }
            }
        }
    }

```
这样就能实现了，但存在一个问题，如果点击绿色区域，及li的区域，li背景色会变改变成红色。
原来target也能拿到父级元素，及li，所以上一段代码还是有bug的，没有考虑完全，可以再加一个筛选
改进：

```javascript
    var grandpa = document.getElementById('grandpa')
    var father1 = document.getElementById('father1')

    grandpa.onclick = function (event) {
        alert(1)
        
        father1.onclick = function (e) {
            var target = e.target
            console.log(target)
            alert(2)
             for (let i = 0; i < this.children.length; i++) {
                if (target !== this.children[i]&&target.tagName !== "LI"){
                    //通过判断target的标签名来得知是否为子元素
                    this.children[i].style.background = "red"
                    
                }else if(target === this.children[i]){
                    target.style.background = "yellow"
                }
                else{}
            }
        }
    }
```
这样点击区域就限定在span里了，但又有一个问题，因为事件会冒泡，即向上传递事件，会触发grandpa的onclick事件，所以可以在father1里加上e.stopPropagation()，用来阻止向上冒泡不去触发alert(1),但又不阻止本身的点击事件。

## 总结

api都有，套用虽然简单，但针对需求如果不能灵活运用，还是写不好代码的，这次修bug对事件委托、冒泡及捕获有了更好的认知。

## bug中的bug
~~（ps:测试过程中，还有一个没解决的bug，即浏览器渲染完成后，点击span元素，虽然阻止了冒泡，但是仍然会触发grandpa的alert（1），自身的事件不被触发，第二次点击之后恢复正常，暂时还没有头绪，解决了再更）~~

因为father1的onclick事件嵌套在grandpa的onclick事件里面，所以第一次渲染的时候，如果不点击把事件触发了，是无法找到father点击事件所对应的那个函数的，只有触发了一次，浏览器才能解析到father的点击事件，将函数存到内存中，这时候点击father里面的元素才能触发。所以是自己写的代码有问题，不过因为需求上必须先点击grandpa，然后才能点击father，即像下拉菜单那种效果，所以真实项目上就忽略了这个bug。


## 补充

这里补充一点就是事件流的顺序。

dom level 2 规定了事件流的三个阶段：

1.事件捕获
2.处于目标阶段（事件处理）
3.冒泡阶段

通过两个例子来弄清楚。

```HTML
<style>
div{
  border: 1px solid black;
  padding: 10px;
}
</style>
<div id="grandpa">
  爷爷
  <div id="father">
    爸爸
    <div id="child">
      儿子
    </div>
  </div>
</div>
```

```javascript
grandpa.addEventListener('click',function f1(){
  console.log('grandpa')
})
father.addEventListener('click',function f2(){
  console.log('father')
})
child.addEventListener('click',function f3(){
  console.log('child')
})
/*
*此时打印出的排列数序为
*
*    child
*    father
*    grandpa
*
*因为事件监听不传第三个参数则默认为false，false则为冒泡，所以是从里到外打印出结果
*/
```

再来一个例子就清晰了。

```javascript
grandpa.addEventListener('click',function f1(){
  console.log('grandpa')
},true) //设置为true为事件捕获
father.addEventListener('click',function f2(){
  console.log('father')
})
child.addEventListener('click',function f3(){
  console.log('child')
})
/*
*此时打印出的排列数序为
*
*    grandpa
*    child
*    father
*
*先打印出grandpa，因为先执行的捕获，而father及child在捕获阶段的函数为false，所以不会执行，等到冒泡阶段才会执行
*/
```

### 例外

如果一个节点同时有捕获及冒泡，这样事件执行顺序按代码的先后来。
```javascript
child.addEventListener('click',function f3(){
  console.log('事件捕获')
},true)

child.addEventListener('click',function f3(){
  console.log('事件冒泡')
})
/*
*此时打印出的排列数序为
*
*    事件捕获
*    事件冒泡
*/

child.addEventListener('click',function f3(){
  console.log('事件冒泡')
})

child.addEventListener('click',function f3(){
  console.log('事件捕获')
},true)

/*
*此时打印出的排列数序为
*
*    事件冒泡
*    事件捕获
*/
```
