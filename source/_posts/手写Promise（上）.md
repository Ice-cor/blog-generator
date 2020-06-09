---
title: 手写Promise（上）
date: 2020-06-09 23:12:12
tags: js Promise
---

Promise对于现在的前端来说并不陌生，在项目中用的也算频繁，但是交流一番后发现很多人只停留在用的基础上，或者说有点经验的也只是了解到能解决回调地狱的问题，但对其原理只是一知半解，所以今天写一下Promise，深入了解一下其原理。

## Promise的优点
1.优化代码的结构，使代码相对清晰可读；<br />2.消灭if(err)的代码；<br />

## Promise的使用
通常在封装ajax的时候，用Promise会比较方便，这里看下最简单的用法。

```javascript
const promise = new Promise((resolve, reject) => {
    if (true) {
        setTimeout(() => {
            resolve(10)
        }, 2000)
    } else {
        reject(20)
    }

})
const success = (result) => {
    console.log('成功了，结果是：' + result) // 两秒后打印出结果 10
}
const fail = (reason) => {
    console.log('失败了，原因是：' + reason)
}
promise.then(success, fail)
```

## Promise的api

首先得知道Promise是一个类；

1. 类方法(all/settled/race/reject/resolve)
1. 对象属性(then/finally/catch)
1. 对象内部属性(state = pendding/fulfilled/rejected)

## 根据A+规范手写Promise

这里通过测试驱动开发的方式去写代码，使用到chai、sinon、sinon-chai的库，用TypeScript来写。
<br />[Promise A+规范](https://promisesaplus.com/)<br />

### 测试与开发


#### 基础测试

- Promise是一个类
- 类拥有 then 方法
- 有三种状态 ( pending | fulfilled | rejected )
- 初始化必须接受一个函数fn
- fn接受两个参数分别为resolve、reject
- resolve及rejecg必须为函数;
- fn立即调用；

```javascript
import * as chai from 'chai';
import * as sinon from 'sinon';
import * as sinonChai from 'sinon-chai';
import Promise, {State} from '../src/promise2';

const assert = chai.assert;

describe('Promise', () => {
    it('是一个类', () => {
        assert.isFunction(Promise);
        assert.isObject(Promise.prototype);
    })
    it('new Promise() 必须接受一个函数', () => {
        assert.throw(() => { // 预测会报错
            // @ts-ignore
            new Promise();
        })
    })
    it('new Promise(fn)的返回值是一个对象，且有then方法', () => {
        const promise = new Promise(() => { })
        assert.isFunction(promise.then)
    })
    it('new Promise(fn), fn立即执行', () => {
        const fn = sinon.fake(); // 创建一个假函数
        new Promise(fn);
        assert.isTrue(fn.called);
    })
    it('new Promise(fn), fn执行时可以接受两个函数resolve、reject', () => {
        new Promise((resolve, reject) => {
            assert.isFunction(resolve);
            assert.isFunction(reject);
        })
    })
})
```

#### 代码1

```javascript
export enum State {
    pending = 'pending',  // 等待
    fulfilled = 'fulfilled', // 成功
    rejected = 'rejected' // 拒绝
}

class Promise {
    state = State.pending // 'pending'|'fulfilled'|'rejected'
    constructor(fn) {
        if(typeof fn !== 'function') {
            throw new Error('fn必须为函数')
        }
        const resolve = () => {

        }
        const reject = () => {

        }
        fn(resolve, reject)
    }
    then() {

    }
}
```

### 2.2 测试then方法

- promise 必须提供 **then** 方法来存取它当前或最终的值或者原因
- promise 的 **then** 方法接收两个参数 (** onFulfilled  | onRejected** )

#### 测试2.2.1~2.2.3

- onFulfilled  | onRejected 不是函数就忽略
- onFulfilled  | onRejected 必须在 promise 完成后才调用
- 把 resolve | reject 传入的值作为 onFulfilled  | onRejected 的第一个参数
- onFulfilled  | onRejected 在 promise 完成前绝不能被调用
- onFulfilled  | onRejected 只能被调用一次
<br />

```javascript

import * as chai from 'chai';
import * as sinon from 'sinon';
import * as sinonChai from 'sinon-chai';
import Promise, { State } from '../src/promise2';

const assert = chai.assert;

describe('then方法 2.2.1~2.2.3', () => {
    it('new Promise(fn), fn执行时可以接受两个函数resolve、reject', () => {
        new Promise((resolve, reject) => {
            assert.isFunction(resolve);
            assert.isFunction(reject);
        })
    })
    it('new Promise(success)中的 success 会被 resolve 调用的时候执行', done => {
        const success = sinon.fake();
        const promise = new Promise((resolve, reject) => {
            assert.isFalse(success.called);
            resolve();
            setTimeout(() => {
                assert.isTrue(success.called)
                done() // 有异步的情况必须手动结束，否则会忽略掉异步代码执行的情况
            })
        })
        promise.then(success);
    })
    it('new Promise(null, fail)中的 fail 会被 reject 调用的时候执行', done => {
        const fail = sinon.fake();
        const promise = new Promise((resolve, reject) => {
            assert.isFalse(fail.called);
            reject();
            setTimeout(() => {
                assert.isTrue(fail.called)
                done()
            })
        })
        promise.then(null, fail);
    })
    it('promise.then(success, fail)，success必须为函数, 否则会被忽略', () => {
        const promise = new Promise((resolve, reject) => {
            resolve()
        })
        promise.then(null, 2);
    })
    it(`2.2.2 
        then里所传的success函数必须在状态变为fulfilled后才执行；
        resolve(arg)中arg的值作为success(arg)中的第一个参数；
        此函数在promise完成(fulfilled)之前绝对不能被调用；
        此函数绝对不能被调用超过一次；
        `,
        (done) => {
            const success = sinon.fake();
            const promise = new Promise((resolve, reject) => {
                assert.isTrue(success.notCalled) // 完成前还未执行
                resolve(1)
                resolve(2) // 多次调用
                setTimeout(() => {
                    assert.isTrue(promise.state === 'fulfilled') // 状态已经改变
                    assert.isTrue(success.called) // promise完成后执行
                    assert(success.calledWith(1)) // 第一个参数为resolve所传的值
                    assert(success.calledOnce) // 只执行了一次
                    done()
                }, 0)
            })
            promise.then(success);
        })
    it(`2.2.3 同上，状态是rejected`,
        (done) => {
            const fail = sinon.fake();
            const promise = new Promise((resolve, reject) => {
                assert.isTrue(fail.notCalled) // 完成前还未执行
                reject(1)
                reject(2) // 多次调用
                setTimeout(() => {
                    assert.isTrue(promise.state === 'rejected') // 状态已经改变
                    assert.isTrue(fail.called) // promise完成后执行
                    assert(fail.calledWith(1)) // 第一个参数为reject所传的值
                    assert(fail.calledOnce) // 只执行了一次
                    done()
                }, 0)
            })
            promise.then(null, fail);
        })
})

```

#### 代码2

```javascript
export enum State {
    pending = 'pending',
    fulfilled = 'fulfilled',
    rejected = 'rejected'
}

class Promise {
    state = State.pending
    succeed = null
    fail = null
    constructor(fn) {
        if (typeof fn !== 'function') {
            throw new Error('fn必须为函数')
        }
        const resolve = (result) => {
            if (this.state !== State.pending) return // 只执行一次succeed
            this.state = State.fulfilled // 改变状态
            setTimeout(() => { // 用异步模拟微任务
                if (typeof this.succeed === 'function') { // 只传函数
                    this.succeed(result) // 将resolve传入值当作第一个参数
                }
            }, 0)
        }
        const reject = (reason) => {
            if (this.state !== State.pending) return
            this.state = State.rejected
            setTimeout(() => {
                if (typeof this.fail === 'function') {
                    this.fail(reason)
                }
            }, 0)
        }
        fn(resolve, reject)
    }
    then(succeed?, fail?) { // 传入参数可选
        if (typeof succeed === 'function') {
            this.succeed = succeed
        }
        if (typeof fail === 'function') {
            this.fail = fail
        }
    }
}

export default Promise
```

#### 测试2.2.4~2.2.6

- 在代码执行完之前(同步代码)，不调用 onFulfilled  | onRejected
- onFulfilled  | onRejected 必须作为函数被调用，且不绑定this的指向
- onFulfilled  | onRejected 回调必须根据最原始的 then 顺序来调用

```javascript
import * as chai from 'chai';
import * as sinon from 'sinon';
import * as sinonChai from 'sinon-chai';
import Promise, { State } from '../src/promise2';

const assert = chai.assert;

describe('then方法 2.2.4~2.2.6', () => {
    it('2.2.4 在代码执行完之前，回调的success函数不得执行',
        (done) => {
            const success = sinon.fake();
            const promise = new Promise((resolve, reject) => {
                resolve();
            })
            promise.then(success);
            assert.isTrue(success.notCalled); // 此处未执行，后面还有代码
            assert(1 === 1);
            setTimeout(() => {
                assert.isTrue(success.called); // 用异步保证是最后执行的
                done();
            }, 0)
        })
    it('2.2.4 在代码执行完之前，回调的fail函数不得执行',
        (done) => {
            const fail = sinon.fake();
            const promise = new Promise((resolve, reject) => {
                reject();
            })
            promise.then(null, fail);
            assert.isTrue(fail.notCalled);
            assert(1 === 1);
            setTimeout(() => {
                assert.isTrue(fail.called);
                done();
            }, 0)
        })
    it('2.2.5 当success、fail必须被当作函数调用，且不绑定this的指向',
        (done) => {
            const promise = new Promise((resolve, reject) => {
                resolve();
            })
            promise.then(function () {
                'use strict'
                assert(this === undefined) // 严格模式下保证this等于undefined
                done()
            });
        })
    it('2.2.6 then可以在同一个promise里多次被调用，且按顺序进行',
        (done) => {
            const promise = new Promise((resolve, reject) => {
                resolve();
            })
            const callbacks = [sinon.fake(), sinon.fake(), sinon.fake()]
            promise.then(callbacks[0])
            promise.then(callbacks[1])
            promise.then(callbacks[2]) // 多次调用then
            setTimeout(() => {
                assert(callbacks[0].called)
                assert(callbacks[1].called)
                assert(callbacks[2].called) // 保证每一个都被调用成功
                assert(callbacks[1].calledAfter(callbacks[0]))
                assert(callbacks[2].calledAfter(callbacks[1])) // 保证调用顺序正确
                done()
            }, 0)
        })
})

```

#### 代码3

```javascript
export enum State {
    pending = 'pending',
    fulfilled = 'fulfilled',
    rejected = 'rejected'
}

class Promise {
    state = State.pending
    callbacks=[] // 代替succeed、fail，供多个then调用
    constructor(fn) {
        if (typeof fn !== 'function') {
            throw new Error('fn必须为函数')
        }
        const resolve = (result) => {
            if (this.state !== State.pending) return
            this.state = State.fulfilled
            setTimeout(() => {
                this.callbacks.forEach(handle => { // 遍历缓存succeed数组
                    if (typeof handle[0] === 'function') {
                        handle[0].call(undefined, result) // 指定this为undefined
                    }
                });
            }, 0)
        }
        const reject = (reason) => {
            if (this.state !== State.pending) return
            this.state = State.rejected
            setTimeout(() => {
                this.callbacks.forEach(handle => {
                    if (typeof handle[1] === 'function') {
                        handle[1].call(undefined, reason)
                    }
                });
            }, 0)
        }
        fn(resolve, reject)
    }
    then(succeed?, fail?) {
        const handle = [] // 用数组缓存保证执行顺序
        if (typeof succeed === 'function') {
            handle[0] = succeed // handle[0]缓存succeed函数，防止多次调用then的情况
        }
        if (typeof fail === 'function') {
            handle[1] = fail // handle[1]缓存fail函数，防止多次调用then的情况
        }
        this.callbacks.push(handle) // 推到callbacks数组中
    }
}

export default Promise
```

## 小结

代码写到这里，可以实现大部分Promise的功能，剩余的留到下一篇来说明实现。
