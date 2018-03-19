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

_（ps:测试过程中，还有一个没解决的bug，即浏览器渲染完成后，点击span元素，虽然阻止了冒泡，但是仍然会触发grandpa的alert（1），自身的事件不被触发，第二次点击之后恢复正常，暂时还没有头绪，解决了再更）_