---
title: 手写Promise & 最近阅读源码的感悟
date: 2022-10-17T22:00:00.000+00:00
lang: zh
duration: 10min
---

通过这段时间的源码学习，比如说vue, unocss，我发现他们的思想极其简单，但做起来就很麻烦

如`vue`,核心工作原理甚至就一句话的事
```ts
effect(() => render(App, '#app'))
```

如`unocss`
```ts
const rules = [
  ['text-red', { color: 'red' }]
]

const generator = (options = {}) => {
  const { rules } = options

  const tokens = getValidTokens()

  return {
    get css() {
      return rules.filter(item => tokens.includes(item[0])).map((rule) => {
        const name = rule[0]
        const content = Object.entires(
          rule[1]).map(([key, val]) => `${key}: ${val};`
        )

        return `.${name} {${content.join('')}}`
      }).join('\n')
    }
  }
}
```


核心原理看着很优美让人感慨，但真正做起来的时候边缘case和特殊情况处理还是挺复杂的

之前看过一点promise原理，当时看了一下基本思想代码，觉得很简单没放在心上，毕竟原理是真的不难，但是今天自己真正上手写了一番后发现还是有不少问题的，花了大概一个小时差不多写好了，上代码吧

```ts
class _Promise {
  static PENDING = 'pending'
  static FULFILLED = 'fulfilled'
  static REJECTED = 'rejected'

  onFulfilledCallbacks = []
  onRejectedCallbacks = []

  constructor(executor) {
    this.PromiseState = _Promise.PENDING
    this.PromiseResult = null
    try {
      executor(this.resolve.bind(this), this.reject.bind(this))
    }
    catch (err) {
      this.reject(err)
    }
  }

  resolve(result) {
    if (this.PromiseState === _Promise.PENDING) {
      this.PromiseState = _Promise.FULFILLED
      this.PromiseResult = result
      for (const fn of this.onFulfilledCallbacks)
        fn(result)

    }
  }

  reject(reason) {
    if (this.PromiseState === _Promise.PENDING) {
      this.PromiseState = _Promise.REJECTED
      this.PromiseResult = reason
      for (const fn of this.onRejectedCallbacks)
        fn(reason)

    }
  }

  then(onFullfieldCallback, onRejectedCallback) {
    const promise = new _Promise((resolve, reject) => {
      const executeCallback = (callback, handler) => {
        queueMicrotask(() => {
          try {
            if (typeof callback !== 'function') {
              handler(this.PromiseResult)
            }
            else {
              const result = callback(this.PromiseResult)
              resolvePromise(promise, result, resolve, reject)
            }
          }
          catch (err) {
            reject(err)
          }
        })
      }

      if (this.PromiseState === _Promise.FULFILLED) {
        executeCallback(onFullfieldCallback, resolve)
      }
      else if (this.PromiseState === _Promise.REJECTED) {
        executeCallback(onRejectedCallback, reject)
      }
      else if (this.PromiseState === _Promise.PENDING) {
        this.onFulfilledCallbacks.push(() => {
          executeCallback(onFullfieldCallback, resolve)
        })
        this.onRejectedCallbacks.push(() => {
          executeCallback(onRejectedCallback, reject)
        })
      }
    })

    return promise
  }
}

function resolvePromise(promise, result, resolve, reject) {
  if (result === promise)
    throw new TypeError('Chaining cycle detected for promise')

  if (result instanceof _Promise) {
    result.then((res) => {
      resolvePromise(promise, res, resolve, reject)
    }, reject)
  }
  else if (
    result !== null
    && (typeof result === 'function' || typeof result === 'object')
  ) {
    // thenable https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise#thenables
    let thenable
    try {
      thenable = result.then
    }
    catch (err) {
      reject(err)
    }

    if (typeof thenable === 'function') {
      let called = false

      try {
        if (called)
          return

        thenable.call(promise, (res) => {
          called = true
          resolvePromise(promise, res, resolve, reject)
        })
      }
      catch (err) {
        if (called)
          return

        reject(err)
      }
    }
    else {
      resolve(result)
    }
  }
  else {
    resolve(result)
  }
}
```
