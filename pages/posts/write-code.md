---
title: 记录一些手写题
date: 2022-10-27T22:00:00.000+00:00
lang: zh
duration: 10min
---


## instanceof
```js
const _instanceof = (obj, type) => {
  const filterTypes = ['string', 'symbol', 'number', 'boolean', 'undefined']
  if (filterTypes.includes(typeof obj))
    return false

  const objProto = type.prototype
  let search = obj.__proto__

  while (true) {
    if (search === null)
      return false

    else if (search === objProto)
      return true

    search = search.__proto__
  }
}
```
