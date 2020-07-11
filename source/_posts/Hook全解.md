---
title: Hook全解
date: 2020-07-11 20:31:42
tags:
---

# Hook全解

Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。
> 引自react官网

在16.8之前，常用的有两种组件：**class组件**、**函数组件**；一般在写不复杂的组件时，会使用函数组件，因为首先书写起来很简单，代码量少；其次函数组件不带有状态，符合函数式的写法；虽然函数组件好用，但如果是较为复杂的组件，只能使用class组件，而class写起来相对又比较繁琐且难以理解，不直观，直到16.8出现很好的解决了这个问题，**Hook就是带有状态的组件。**

## 简单示例

一个简单的Hook书写例子：

```javascript

import React, { useState } from 'react';

function Example() {
  // 声明一个新的叫做 “count” 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}> // 点击按钮count依次 +1
        Click me
      </button>
    </div>
  );
}

```

## useState

通过手写简单了解下其工作原理：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

let _state; // 创建一个变量state存储当前组件的状态
const myUseState = (initialValue) => {
  _state = _state === undefined ? initialValue : _state
  const _setState = (newState) => {
    _state = newState;
    render()
  }
  return [_state, _setState]
}

const render = () => { // 粗暴的表示render方法，当然react不是这么渲染的
  ReactDOM.render(
    <App />,
    document.getElementById('root')
  );
}

function App() {
  const [count, setCount] = myUseState(0)
  return (
    <div>
      <span>{count}</span>
      <button onClick={() => {setCount(count + 1)}}>click</button>
    </div>
  )
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

原理很简单，创建一个变量存储当前的组件的状态，在渲染前把新的状态更新即可。

### 多个useState实现

上面写的只支持一个useState，如果多个会被覆盖，如果考虑多个，也很简单，用数组去存放所有state状态即可。

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

let _state = []; // 用数组去存放所有state
let index = 0; // 每个state的初始化
const myUseState = (initialValue) => {
  let currentIndex = index;
  _state[currentIndex] = _state[currentIndex] === undefined ? initialValue : _state[currentIndex]
  const _setState = (newState) => {
    _state[currentIndex] = newState;
    render()
  }
  index += 1
  return [_state[currentIndex], _setState]
}


const render = () => {
  index = 0;
  ReactDOM.render(
    <App />,
    document.getElementById('root')
  );
}

function App() {
  const [count, setCount] = myUseState(0)
  const [count2, setCount2] = myUseState(0)
  return (
    <div>
      <div>
        <span>{count}</span>
        <button onClick={() => {setCount(count + 1)}}>click</button>
      </div>
      <div>
        <span>{count2}</span>
        <button onClick={() => {setCount2(count2 + 1)}}>click</button>
      </div>
    </div>
  )
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

这样即使多个state也能实现。

### 为什么useState使用需要保证顺序

react官网上强调，避免在条件语句里面使用Hook，因为要保证调用的顺序；但是为什么要保证调用顺序呢？
如果只从字面上去猜很难理解，根据上面我们手写的代码不难看出：**因为存储state状态的变量是一个数组，只有保证顺序调用才能拿到对应state正确的值！**
所以react官网上才会强调要按顺序使用。


### state存在分身的情况

先看下面代码：

```javascript
const Hook = () => {
    const [count, setCount] = useState(0);
    const log = () => {
        setTimeout(() => {
            console.log(count)
        }, 3000)
    }
    return (
        <div>
            <span>{count}</span>
            <button onClick={() => {setCount(count + 1)}}>click me</button>
            <button onClick={log}>log</button>
        </div>
    )
}
```
先点击 **click me **按钮，再点击 **log** 按钮，结果是三秒后会打印出 1 。如果先点击 **log** 按钮，再点击 **click me** 按钮，三秒后会打印出 0 ，页面显示 1 ，为什么会出现这样的情况？

1. setCount执行的时候，会重新执行Hook这一函数，函数里面的count会被重新赋值。
2. log 函数里的count是Hook函数执行前的值， 所以执行前和执行后的值不是同一个值。

如果需要保证值是同一个引用，可以使用下列的两种方法：useRef和useContext。

#### useRef

使用很简单 `const refContainer = useRef(initialValue);` ，其实useRef就是创建了一个对象`{current: ...}`，一般是用来保存dom节点，其实它的功能不仅仅于此，它可以保存任意的值，且不受节点更新影响，不好的地方就是值改变不会自动触发render，需要手动render才会使页面改变。

#### useContext

使用方法：

```javascript
// 摘自React官网示例

const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // 此处能拿到顶层组件传下来的值
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

useContext的好处，是避免了每层组件传递props，方便书写。

### 使用useState注意事项
1.如果state是一个对象，是否能局部更新？

```javascript
// hook

const Hook = () => {
    const [user, setUser] = useState({name:'Jack', age: 18});
    return (
        <div>
            <span>{user.name}</span>
						<span>{user.age}</span>  /** 点击按钮后会消失，即undefined*/
            <button onClick={()=>{setUser({name: 'Tom'})}}>log</button>
        </div>
    )
}
```

上面有说到，其实是state的重新赋值，所以会被新对象重新替换，不会对进行局部更新，如果只需更新对象内的某些值，只能使用展开符 `setUser({...user, name: 'Tom'})` 。
2.对象的引用地址相同，hook不会更新。

```javascript
// hook

const Hook = () => {
    const [user, setUser] = useState({name:'Jack', age: 18});
  	const onClick = () => {
      name.name = 'Tom'
    	setUser(user);
    }
    return (
        <div>
            <span>{user.name}</span>
						<span>{user.age}</span>  
     				 /** 此时点击按钮后不会发生任何变化 */
            <button onClick={onClick}>log</button>
        </div>
    )
}
```

因为Hook不会去检查对象内部变化情况，只针对引用地址作分析，所以如果地址不变，Hook不会去执行，如果需要使更新执行，那就必须改变引用地址，即赋值一个新对象，向第一个注意事项说到的那样。

## useEffect
简单来说就是函数的副作用，因为以前的React的Function组件是没有生命周期的，Hook中的useEffect就是用来补足这一块，这就是使得函数组件能发扬光大的原因。
用法：

```javascript
import React, { useState, useEffect } from 'react'

// 注意：useEffects 一定是在浏览器渲染完成后才执行
const useEffects = () => {
    const [count, setCount] = useState(0);
  	// 相当于 componentDidMount
    useEffect(()=>{
        console.log('首次渲染后执行，之后不执行')
    }, [])
  	// 相当于 componentDidUpdate
    useEffect(()=>{
        console.log('根据count的是否更新执行，首次渲染也会执行')
    }, [count])
  	// 相当于 componentWillUnmount
    useEffect(()=>{
        return () => { // 返回一个函数
            console.log('卸载之前执行')
        }
    })
    // 第二个参数不传值
    useEffect(() => {
        console.log('除了卸载，任意时候都会执行')
    })
    return (
        <div>
            <div>{count}</div>
            <button onClick={()=>{setCount(count + 1)}}>+1</button>
        </div>
    )
}

export default useEffects;
```

## useLayoutEffect

这一个API跟useEffect很类似，只不过是执行顺序有点差别而已。
用法：

```javascript
import React, { useState, useLayoutEffect } from 'react'

const useLayoutEffects = () => {
    const [count, setCount] = useState(0);
  	// 相当于 componentDidMount
    useEffect(()=>{
        console.log('首次渲染后执行，之后不执行')
    })
  	useLayoutEffects(()=>{
    		console.log('首次渲染后执行，在渲染前执行')
    })
    return (
        <div>
            <div>{count}</div>
            <button onClick={()=>{setCount(count + 1)}}>+1</button>
        </div>
    )
}

export default useLayoutEffects;
```

**那为什么需要这个API呢？**顾名思义，与useEffect只差Layout，也就是说，这个API最好是在有改变Dom的时候执行。
例子：

```javascript
// hook

const Hook = () => {
    const [user, setUser] = useState({name:'Jack', age: 18});
  	useEffect(()=>{
    	const name = document.getElementById('name');
      name.innerText = 'Marry';
    })
    return (
        <div id="app">
            <span id="name">{user.name}</span>
						<span>{user.age}</span>  /** 点击按钮后会消失，即undefined*/
            <button onClick={()=>{setUser({name: 'Tom'})}}>log</button>
        </div>
    )
}
```

上面的代码，初始值的name是 "Jack" ，等浏览器渲染完后，会执行 useEffect，把 span 内的文本改变成为 "Marry"，其实如果渲染较慢，会有一个闪烁的情况，对用户体验不是很好，而使用 useLayoutEffect 就能避免这种情况，因为是渲染前执行，所以用户只能看到一次 "Marry"。
**是不是全都使用 useLayoutEffect更好？**
一般情况，改变dom操作比较少，所以常规操作放到渲染后才执行触发比较好，只有遇到需要改变dom操作时用它才符合这个API的理念。


## useMemo

这个API一般配合`React.memo`一起使用，检查props的更新来决定是否重新渲染，避免不必要渲染的情况，可以优化性能，但因为单纯的使用`React.memo`还是有bug，当props传递函数的时候，是一定会更新子组件的，而使用 useMemo就能解决这种状况。
用法：

```javascript
export default function App() {
  console.log("app");
  const [n, setN] = React.useState(0);
  const [m, setM] = React.useState(0);
  const onClickChild = React.useMemo(() => {
    console.log(m);
  }, [m]); // 只有m有变化的时候才会更新子组件
  // 如果onClickChild = () => {}，每次app组件点击按钮的时候都会更新子组件
  return (
    <div className="App">
      <h2>{n}</h2>
      <button
        onClick={() => {
          setN(n + 1);
        }}
      >
        click
      </button>
      <Child2 data={m} onClick={onClickChild} />
    </div>
  );
}

function Child(props) {
  console.log("child");
  return (
    <div>
      <span>{props.data.name}</span>
      <button onClick={props.onClickChild}>child click</button>
    </div>
  );
}

const Child2 = React.memo(Child);
```

## useReducer
如果使用过redux会很熟悉，就是做状态管理的，使用这个API基本上已经可以取代redux了。
示例：
```javascript
// reducer.jsx
export function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

// other.jsx
import {reducer, initialState} from './reducer'

const Child1 = () => {
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
        <div>
            <div>
                <span>{state.count}</span>
                <button onClick={() => { dispatch({type: 'increment'}) }}>click +</button>
            </div>
        </div>
    )
}

const Child2 = () => {
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
        <div>
            <div>
                <span>{state.count}</span>
                <button onClick={() => { dispatch({type: 'increment'}) }}>click +</button>
            </div>
        </div>
    )
}
```
如果需要改变同一个值，需要配合useContext使用。

## 自定义Hook

自定义Hook，主要作用就是提取公共的逻辑方便各个组件使用。

示例：

```javascript
// useList.jsx（自定义Hook）
const useList = (initialValue) => {
  const [list, setList] = useState(initialValue);
  useEffect(() => {
    ajax("/list").then(list => {
      setList(list);
    });
  }, []);
  return {
    list: list
      ? list.map(item => {
          return <li key={item.id}>{item.name}</li>;
        })
      : "加载中...",
    setList
  };
};

function ajax(url) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([
        { id: 1, name: "Jack" },
        { id: 1, name: "Tom" },
        { id: 1, name: "Marry" }
      ]);
    }, 2000);
  });
}

export default useList;

// 其他组件 app.jsx
import useList from "./useList";

export default function App() {
  const { list, setList } = useList(null); 
  return (
    <div className="App">
      <ol>{list}</ol>
      <button
        onClick={() => {
          setList([{ id: 1, name: "123" }]);
        }}
      >
        setList
      </button>
    </div>
  );
}

```

其实就是看你怎么熟练去运用函数而已，怎样写都没问题。

[原文](https://www.yuque.com/u283460/kb/twygc6)