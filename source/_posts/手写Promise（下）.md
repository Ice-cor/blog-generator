---
title: 手写Promise（下）
date: 2020-06-11 20:42:00
tags:
---

之前手写Promise完成了大部分功能，剩余的因为比较繁琐，所以放到了这一部分详细讲解。

## 微任务

之前一直使用setTimeout去模拟微任务操作，因为兼容性好；但是原生Promise其实是微任务，优先级会比宏任务定时器要高，所以后面会使用微任务去实现。

`process.nextTick(()=>{})`

本来想用setImmediate代替setTimeout的，后来发现依旧有时间差的问题，如果与 `setTimeout(fn, 0)` 做比较，不一定优先执行，所以选择更快的 **nextTick。**
**[Node 事件循环解析链接](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)**

`new MutationObserver(fn)`

这个是符合浏览器环境可以执行的微任务，大概意思是在监听某个dom变化后执行函数。

```javascript
function nextTick(fn) {
  	// node 环境
    if (process !== undefined && typeof process.nextTick === "function") {
        return process.nextTick(fn);
    } else {
      // 浏览器环境
        var counter = 1;
        var observer = new MutationObserver(fn);
        var textNode = document.createTextNode(String(counter));

        observer.observe(textNode, {
            characterData: true
        });

        counter = counter + 1;
        textNode.data = String(counter);
    }
}
```

## 测试2.2.7
then必须返回一个Promise

#### 测试2.2.7.1

- 2.2.7.1 如果onFulfilled或onRejected返回一个值x, 运行 [[Resolve]](promise2, x) 
- 2.2.7.2 如果onFulfilled或onRejected抛出一个异常e，promise2必须被拒绝（rejected）并把e当作原因
**（这里会关联到 2.3 Promise解决程序）**

#### 2.3 Promise解决程序  [[Resolve]](promise2, x) 

- 2.3.1 如果promise和x引用同一个对象，则用TypeError作为原因拒绝（reject）promise


- 2.3.2 如果x是一个promise, 采用promise的状态
- 2.3.3 如果x是一个对象或方法的若干规定
- 2.3.4 如果x既不是对象也不是函数，用x完成(fulfilled)promise

```javascript
import * as chai from 'chai';
import * as sinon from 'sinon';
import * as sinonChai from 'sinon-chai';
import Promise from '../src/promise';

const assert = chai.assert;

describe('then方法 2.2.7', () => {
    it('2.2.7 then必须返回一个promise',
        (done) => {
            const promise1 = new Promise((resolve, reject) => {
                resolve();
            })
            const promise2 = promise1.then();
            setTimeout(() => {
                assert(promise2 instanceof Promise)
                done()
            }, 0)
        })
    it('2.2.7.1 如果onFulfilled或onRejected返回一个值x，运行 [[Resolve]](promise2, x)',
        (done) => {
            const promise1 = new Promise((resolve, reject) => {
                resolve();
            })
            const promise2 = promise1.then(() => '成功');
            promise2.then((result) => {
                assert(result === '成功');
                done();
            })
        })
   it('2.2.7.1 如果onFulfilled或onRejected返回一个值x，运行 [[Resolve]](promise2, x)',
        (done) => {
            const promise1 = new Promise((resolve, reject) => {
                resolve();
            })
            const promise2 = promise1.then(() => '成功');
            promise2.then((result) => {
                assert(result === '成功');
                done();
            })
        })
    it('2.2.7.1 onFulfilled的返回值x是一个Promise的实例',
        (done) => {
            const fn = sinon.fake()
            const promise1 = new Promise((resolve, reject) => {
                resolve();
            })
            const promise2 = promise1.then(() => new Promise((resolve) => { resolve() })); // 返回一个promise实例
            promise2.then(fn)
            setTimeout(() => {
                assert(fn.called)
                done()
            }, 0)
        })
    it('2.2.7.1 onFulfilled的返回值x是一个Promise的实例，且失败',
        (done) => {
            const fn = sinon.fake()
            const promise1 = new Promise((resolve, reject) => {
                resolve();
            })
            const promise2 = promise1.then(() => new Promise((undefined, reject) => { reject() })); // 返回一个promise实例
            promise2.then(null, fn)
            setTimeout(() => {
                assert(fn.called)
                done()
            }, 0)
        })
    it('2.2.7.1 onRejected返回的x是一个Promise的实例',
        (done) => {
            const fn = sinon.fake()
            const promise1 = new Promise((resolve, reject) => {
                reject();
            })
            const promise2 = promise1.then(
                null, 
                () => new Promise((resolve, reject) => resolve())); // 返回一个promise实例
            promise2.then(fn)
            setTimeout(() => {
                assert(fn.called)
                done()
            }, 0)
        })
    it('2.2.7.1 onRejected返回的x是一个Promise的实例, 且失败',
        (done) => {
            const fn = sinon.fake()
            const promise1 = new Promise((resolve, reject) => {
                reject();
            })
            const promise2 = promise1.then(
                null, 
                () => new Promise((resolve, reject) => reject())); // 返回一个promise实例
            promise2.then(null, fn)
            setTimeout(() => {
                assert(fn.called)
                done()
            }, 0)
        })
   it('2.2.7.2 如果onFulfilled抛出异常, promise2 必须被拒绝（rejected）并把e当作原因',
        (done) => {
            const fn = sinon.fake()
            const error = new Error('抛出异常')
            const promise1 = new Promise((resolve, reject) => {
                resolve();
            })
            const promise2 = promise1.then(() => {
                throw error
            })
            promise2.then(null, fn)
            setTimeout(() => {
                assert(fn.called)
                assert(fn.calledWith(error))
                done()
            }, 0)
        })
    it('2.2.7.2 如果onRejected抛出异常, promise2 必须被拒绝（rejected）并把e当作原因',
        (done) => {
            const fn = sinon.fake()
            const error = new Error('抛出异常')
            const promise1 = new Promise((resolve, reject) => {
                reject();
            })
            const promise2 = promise1.then(null, () => {
                throw error
            })
            promise2.then(null, fn)
            setTimeout(() => {
                assert(fn.called)
                assert(fn.calledWith(error))
                done()
            }, 0)
        })
})
```

#### 代码

```javascript
class Promise {
    resolve(result) {
        if (this.state === 'fulfilled') {
            return;
        }
        this.state = 'fulfilled';
        nextTick(() => { // 微任务 node环境
            this.callbacks.forEach(handle => {
                if (typeof handle[0] === 'function') { // 判断必须放到定时器里面，否则会提前判断，拿到是null的值
                    let x
                    try {
                        x = handle[0].call(undefined, result);
                    } catch (e) {
                        return handle[2].reject(e) // 2.2.7.2 如果抛出异常
                    }
                    handle[2].resolveWith(x);
                }
            })
        })
    }
    reject(reason) {
        if (this.state === 'rejected') {
            return;
        }
        this.state = 'rejected';
        nextTick(() => {
            this.callbacks.forEach(handle => {
                if (typeof handle[1] === 'function') {
                    let x;
                    try {
                        x = handle[1].call(undefined, reason);
                    } catch (e) {
                        handle[2].reject(e)
                    }
                    handle[2].resolveWith(x)
                }
            })
        })
    }
    callbacks = []
    state = 'pending' // 'pending'|'fulfilled'|'rejected'三种状态，初始为'pending'
    constructor(fn) {
        if (typeof fn !== 'function') {
            throw new Error('必须接受一个函数')
        }
        fn(this.resolve.bind(this), this.reject.bind(this));
    }
    then(success?, fail?) {
        const handle = []
        if (typeof success === 'function') {
            handle[0] = success
        }
        if (typeof fail === 'function') {
            handle[1] = fail
        }
        handle[2] = new _Promise(() => { })
        this.callbacks.push(handle)
        return handle[2]
    }
    resolveWith(x) {
        if (this === x) { // 2.3.1
            this.reject(new TypeError('不能是同一个引用对象'))
        }
        if (x instanceof _Promise) { // 2.3.2
            x.then((result) => { // 2.3.3.3
                this.resolve(result) // 2.3.3.3.1 
            }, (reason) => {
                this.reject(reason)  // 2.3.3.3.2 
            })
        }
        if (x instanceof Object) { // 2.3.3
            let then
            try {
                then = x.then // 2.3.3.1 让x作为x.then
            } catch (e) {
                this.reject(e) // 2.3.3.2
            }
            if (typeof then === 'function') { // 2.3.3.3, 如果为函数则调用then
                try {
                    x.then( // 
                        (y) => {
                            this.resolveWith(y) // 2.3.3.3.1
                        }, (r) => {
                            this.resolveWith(r) // 2.3.3.3.2
                        })
                } catch (e) { // 2.3.3.3.4 
                    this.reject(e)
                }
            } else { // 2.3.3.4 如果不是函数，resolve
                this.resolve(x)
            }
        } else { // 2.3.4 如果不是对象，直接resolve
            // 不考虑无限递归的情况
            this.resolve(x)
        }
    }
}
class _Promise {
    resolve(result) {
        if (this.state === 'fulfilled') {
            return;
        }
        this.state = 'fulfilled';
        nextTick(() => { // 微任务 node环境
            this.callbacks.forEach(handle => {
                if (typeof handle[0] === 'function') { // 判断必须放到定时器里面，否则会提前判断，拿到是null的值
                    let x
                    try {
                        x = handle[0].call(undefined, result);
                    } catch (e) {
                        return handle[2].reject(e) // 2.2.7.2 如果抛出异常
                    }
                    handle[2].resolveWith(x);
                }
            })
        })
    }
    reject(reason) {
        if (this.state === 'rejected') {
            return;
        }
        this.state = 'rejected';
        nextTick(() => {
            this.callbacks.forEach(handle => {
                if (typeof handle[1] === 'function') {
                    let x;
                    try {
                        x = handle[1].call(undefined, reason);
                    } catch (e) {
                        handle[2].reject(e)
                    }
                    handle[2].resolveWith(x)
                }
            })
        })
    }
    callbacks = []
    state = 'pending' // 'pending'|'fulfilled'|'rejected'三种状态，初始为'pending'
    constructor(fn) {
        if (typeof fn !== 'function') {
            throw new Error('必须接受一个函数')
        }
        fn(this.resolve.bind(this), this.reject.bind(this));
    }
    then(success?, fail?) {
        const handle = []
        if (typeof success === 'function') {
            handle[0] = success
        }
        if (typeof fail === 'function') {
            handle[1] = fail
        }
        handle[2] = new _Promise(() => { })
        this.callbacks.push(handle)
        return handle[2]
    }
    resolveWith(x) {
        if (this === x) { // 2.3.1
            this.reject(new TypeError('不能是同一个引用对象'))
        }
        if (x instanceof _Promise) { // 2.3.2
            x.then((result) => { // 2.3.3.3
                this.resolve(result) // 2.3.3.3.1 
            }, (reason) => {
                this.reject(reason)  // 2.3.3.3.2 
            })
        }
        if (x instanceof Object) { // 2.3.3
            let then
            try {
                then = x.then // 2.3.3.1 让x作为x.then
            } catch (e) {
                this.reject(e) // 2.3.3.2
            }
            if (typeof then === 'function') { // 2.3.3.3, 如果为函数则调用then
                try {
                    x.then( // 
                        (y) => {
                            this.resolveWith(y) // 2.3.3.3.1
                        }, (r) => {
                            this.resolveWith(r) // 2.3.3.3.2
                        })
                } catch (e) { // 2.3.3.3.4 
                    this.reject(e)
                }
            } else { // 2.3.3.4 如果不是函数，resolve
                this.resolve(x)
            }
        } else { // 2.3.4 如果不是对象，直接resolve
            // 不考虑无限递归的情况
            this.resolve(x)
        }
    }
}

function nextTick(fn) {
    if (process !== undefined && typeof process.nextTick === "function") {
        return process.nextTick(fn);
    } else {
        var counter = 1;
        var observer = new MutationObserver(fn);
        var textNode = document.createTextNode(String(counter));

        observer.observe(textNode, {
            characterData: true
        });

        counter = counter + 1;
        textNode.data = String(counter);
    }
}

export default Promise;

```

#### 小结

这样就基本完成Promise的功能，下面做一些优化的工作。

## 代码重构

```javascript
class Promise {
    callbacks = []
    state = 'pending' // 'pending'|'fulfilled'|'rejected'三种状态，初始为'pending'
    private resolveOrReject(data, state, i) {
        if (this.state !== 'pending') return;
        this.state = state;
        nextTick(() => { // 微任务 node环境
            this.callbacks.forEach(handle => {
                if (typeof handle[i] === 'function') { // 判断必须放到定时器里面，否则会提前判断，拿到是null的值
                    let x
                    try {
                        x = handle[i].call(undefined, data);
                    } catch (e) {
                        return handle[2].reject(e) // 2.2.7.2 如果抛出异常
                    }
                    handle[2].resolveWith(x);
                }
            })
        })
    }
    resolve(result) {
        this.resolveOrReject(result, 'fulfilled', 0);
    }
    reject(reason) {
        this.resolveOrReject(reason, 'rejected', 1);
    }
    constructor(fn) {
        if (typeof fn !== 'function') {
            throw new Error('必须接受一个函数')
        }
        fn(this.resolve.bind(this), this.reject.bind(this));
    }
    then(success?, fail?) {
        const handle = []
        if (typeof success === 'function') {
            handle[0] = success
        }
        if (typeof fail === 'function') {
            handle[1] = fail
        }
        handle[2] = new _Promise(() => { })
        this.callbacks.push(handle)
        return handle[2]
    }
    resolveWith(x) {
        if (this === x) { // 2.3.1
            this.resolveWithSelf()
        }
        if (x instanceof _Promise) { // 2.3.2
            this.resolveWithPromise(x)
        }
        if (x instanceof Object) { // 2.3.3
            this.resolveWithObject(x)
        } else { // 2.3.4 如果不是对象，直接resolve
            // 不考虑无限递归的情况
            this.resolve(x)
        }
    }
    private resolveWithSelf() {
        this.reject(new TypeError('不能是同一个引用对象'));
    }
    private resolveWithPromise(x) {
        x.then(
            result => {
                this.resolve(result);
            },
            reason => {
                this.reject(reason);
            }
        );
    }
    private getThen(x) {
        let then;
        try {
            then = x.then;
        } catch (e) {
            return this.reject(e);
        }
        return then;
    }
    private resolveWithThenable(x) {
        try {
            x.then(
                y => {
                    this.resolveWith(y);
                },
                r => {
                    this.reject(r);
                }
            );
        } catch (e) {
            this.reject(e);
        }
    }
    private resolveWithObject(x) {
        let then = this.getThen(x);
        if (then instanceof Function) {
            this.resolveWithThenable(x);
        } else {
            this.resolve(x);
        }
    }
}
function nextTick(fn) {
    if (process !== undefined && typeof process.nextTick === "function") {
        return process.nextTick(fn);
    } else {
        var counter = 1;
        var observer = new MutationObserver(fn);
        var textNode = document.createTextNode(String(counter));

        observer.observe(textNode, {
            characterData: true
        });

        counter = counter + 1;
        textNode.data = String(counter);
    }
}

export default Promise;
```

## 总结

手写一遍Promise，确实加深了对Promise的理解，还有就是测试驱动开发是真的舒服，只要写好了测试用例，代码怎么修改也不用担心出错，这也是我未来要进行的一个方向。
