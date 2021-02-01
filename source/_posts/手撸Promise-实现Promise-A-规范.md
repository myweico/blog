---
title: '手撸Promise,实现Promise A+规范'
date: 2021-02-02 00:10:23
tags:
- JavaScript

categories:
- JavaScript
---

Promise 是 JavaScript 中描述了异步操作过程，是解决 JavaScript 回调地狱的一个很好的途径。也是我们开发经常用到的东西。

为了更深入的了解一下 Promise，这篇文章就让我们来手写一下 Promise 以及实现es6中常用的api，其实现要求符合 Promise A+ 规范。

<!-- more -->

## Promise A+ 规范
原文地址：https://promisesaplus.com/

## 代码实现
```js
const PENDING = "PNEDING";
const FULFILLED = "FULFILLED";
const REJECTED = "REJECTED";

class Promise {
  constructor(task) {
    // promise 有三个状态，pending、fulfilled、rejected，
    // 状态只能改变一次，即 pending => fulfilled 或者 pending => rejected
    this.status = PENDING;
    // 若状态变更为 fulfilled，promise 需要返回一个 value
    this.value = "";
    // 若状态变更为 rejected，promise 需要返回一个 reason
    this.reason = "";
    // 完成状态的回调参数，当 promise 状态变为 fulfilled 的时候，必须按照注册的顺序执行
    this.fulfill_callbacks = [];
    // rejected状态的回调函数，当 promise 状态变为 rejected 的时候，按照注册的顺序执行
    this.reject_callbacks = [];

    // 初始化的时候，就执行传入的任务，传入一个 resolve 和 reject 的函数
    try {
      task(this._resolve, this._reject);
    } catch (err) {
      // 执行过程中，发生错误，直接 reject
      this._reject(err);
    }
  }

  // 将 promise 改为 resolve 状态的函数
  _resolve(value) {
    // 状态只能改变一次
    if (this.status !== PENDING) return;

    // resolve 应该是放到 微任务队列里面，但是无法实现，只能模拟放到 宏任务队列里面
    setTimeout(() => {
      // 将 promise 改为 fulfilled 状态
      this.status = FULFILLED;

      // 更新 promise 的值
      this.value = value;

      // 按顺序执行 fulfilled 的回调
      this.fulfill_callbacks.forEach((cb) => cb(value));
    }, 0);
  }

  _reject(reason) {
    // 状态只能更改一次
    if (this.status !== PENDING) return;

    // 将 resolve 处理放置到事件队列里面
    setTimeout(() => {
      // 将 promise 状态更改为 rejected
      this.status = REJECTED;

      // 更新 promise 的 reason
      this.reason = reason;

      // 按顺序执行 rejected 的回调
      this.reject_callbacks.forEach((cb) => cb(reason));
    }, 0);
  }

  // promise 必须提供一个 then 方法获取其值或者原因
  // then 接受两个参数, onFulfilled 以及 onRejected
  then(onFulfilled, onRejected) {
    // then函数也是返回一个 promise
    if (this.state === FULFILLED) {
      // promise 状态已经是 fulfilled 状态，
      // 若 onFulfileed 不是函数
      let promise2 = new Promise((resolve, reject) => {
        try {
          // 放置到事件循环处理
          setTimeout(() => {
            if (!this._isFunction(onFulfilled)) {
              // 直接 resolve promise1的值
              resolve(this.value);
            } else {
              // 若是函数，交给 resolvePromise 处理，参数是 promise2 以及 onFulFilled执行的结果
              this._resolvePromise(promise2, onFulfilled(this.value));
            }
          }, 0);
        } catch (err) {
          reject(err);
        }
      });
      return promise2;
    } else if (this.state === REJECTED) {
      let promise2 = new Promise((resolve, reject) => {
        try {
          setTimeout(() => {
            if (!this._isFunction(onRejected)) {
              // onRejected 不是函数，用当前的promise的reason reject then的promise
              reject(this.reason);
            } else {
              // 交给 _resovlePromsie 处理，参数是 then函数的 promise 以及 onRejected 处理的结果
              this._resolvePromise(promise2, onRejected(this.reason));
            }
          }, 0);
        } catch (err) {
          reject(err);
        }
      });
      return promise2;
    } else {
      // 若状态还是 pending，加入到任务队列里面去（观察者模式），当其resolve 或者 reject的时候，将会执行
      let promise2 = new Promise((resolve, reject) => {
        this.fulfill_callbacks.push(() => {
          // 再处理上面 status === FULFILLED 的逻辑
          try {
            setTimeout(() => {
              if (!_isFunction(onFulfilled)) {
                resolve(this.value);
              } else {
                this._resolvePromise(promise2, onFulfilled(this.value));
              }
            }, 0);
          } catch (err) {
            reject(err);
          }
        });
        this.reject_callbacks.push(() => {
          // 在处理上面 status === REJECTED 相同的逻辑
          try {
            setTimeout(() => {
              if (!this._isFunction(onRejected)) {
                reject(this.reason);
              } else {
                this._resolvePromise(promise2, onRejected(this.reason));
              }
            }, 0);
          } catch (err) {
            reject(err);
          }
        });
      });
      return promise2;
    }
  }

  catch = (onRejected) => this.then(null, onRejected);

  finally = (onFinished) => this.then(onFinished, onFinished);

  static deffered() {
    let deffered = {};
    deffered.promise = new Promise((resolve, reject) => {
      deffered.resolve = resolve;
      deffered.reject = reject;
    });
    return deffered;
  }

  static resolve(value) {
    // 若 value 是 Promise 直接返回 Promise
    if (value instanceof Promise) return value;
    return new Promise((resolve) => {
      resolve(value);
    });
  }

  static reject(reason) {
    return new Promise((resolve, reject) => {
      reject(reason);
    });
  }

  static all(promiseList) {
    return new Promise((resolve, reject) => {
      let _count = 0;
      let _responses = [];
      if (promiseList.length === 0) return resolve([]);
      promiseList.forEach((promise, index) => {
        Promise.resolve(promise)
          .then((res) => {
            _responses[index] = res;
            _count = _count + 1;
            if (_count === promiseList.length) {
              //  全部完成了
              resolve(_responses);
            }
          })
          .catch((reason) => {
            // 发生错误了
            reject(reason);
          });
      });
    });
  }

  static race(promiseList) {
    return new Promise((resolve, reject) => {
      if (promiseList.length === 0) return resolve();
      promiseList.forEach((promise) => {
        Promise.resolve(promise)
          .then((res) => {
            resolve(res);
          })
          .catch((err) => {
            reject(err);
          });
      });
    });
  }

  static allSettle(promiseList) {
    return new Promise((resolve, reject) => {
      if (promiseList.length === 0) return resolve([]);
      let _count = 0;
      let _responses = [];
      promiseList.forEach((promise, index) => {
        Promise.resolve(promise)
          .then((value) => {
            _responses[index] = { status: "fulfilled", value: value };
          })
          .catch((reason) => {
            _responses[index] = { status: "rejected", reason: reason };
          })
          .finally(() => {
            _count = _count + 1;
            if (_count === promiseList.length) {
              resolve(_responses);
            }
          });
      });
    });
  }

  static any(promiseList) {
    return new Promise((resolve, reject) => {
      if (promiseList.length === 0) return resolve();
      let _catchCount = 0;
      const _catchReasons = [];

      promiseList.forEach((promise, index) => {
        Promise.resolve(promise)
          .then((value) => {
            resolve(value);
          })
          .catch((reason) => {
            _catchCount = _catchCount + 1;
            _catchReasons[index] = reason;
            if (_catchCount === promiseList.length) {
              reject(_catchReasons);
            }
          });
      });
    });
  }

  _isFunction = (f) =>
    Object.prototype.toString.call(f).toLowerCase() === "[object function]";

  _isObject = (o) =>
    Object.prototype.toString.call(o).toLowerCase() === "[object object]";

  _resolvePromise = (promise, x) => {
    // promise 和 x 指向相同的值，抛出一个 TypeError 作为拒绝的原因
    if (promise === x) {
      promise._reject(new TypeError("promise and x can not be the same"));
    }

    // 若 x 是一个 Promise，promise 需要等待 x-Promise 状态改变
    if (x instanceof Promise) {
      if (x.status === FULFILLED) {
        promise._resolve(x.value);
      } else if (x.status === REJECTED) {
        promise._reject(x.reason);
      } else {
        // x 还在 pending，promise 需要等待 x 的状态
        x.then(
          (value) => {
            this._resolvePromise(promise, value);
          },
          (reason) => {
            promise._reject(reason);
          }
        );
      }
      return;
    }

    if (this._isFunction(x) || this._isObject(x)) {
      let then;
      try {
        then = x.then;
      } catch (err) {
        promise_reject(err);
        return;
      }

      if (this._isFunction(then)) {
        let called = false;
        try {
          then.call(
            x,
            (value) => {
              if (called) return;
              called = true;
              this._resolvePromise(promise, value);
            },
            (reason) => {
              if (called) return;
              called = true;
              promise._reject(reason);
            }
          );
        } catch (err) {
          promise._reject(err);
        }
      } else {
        // then 不是函数
        promise._resolve(x);
      }
    } else {
      // x 不是函数或者对象
      promise._resolve(x);
    }
  };
}
```
