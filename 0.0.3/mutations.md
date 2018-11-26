# 数组 类型的 mutations

在 todomvc 示例中，传入 vuex 的 mutations 是一个Object对象，但是在 shopping-cart 示例中，由于 state 被拆分成了两个 module ，传入给 vuex 构造函数的 mutatios 类型变为了一个数组：

```js
import { cartMutations } from './stores/cart'
import { productsMutations } from './stores/products'

export default new Vuex({
  mutations: [cartMutations, productsMutations]
})
```

那么，这种情况 vuex 源码是如何处理的呢？

## 源码中的 mutations

```js
    // mutations
    this._mutations = Array.isArray(mutations)
      ? mergeObjects(mutations, true)
      : mutations
```

针对数组类型的 mutations ， vuex 在处理的时候将其合并为一个对象。

```js
function mergeObjects (arr, allowDuplicate) {
  return arr.reduce((prev, obj) => {
    Object.keys(obj).forEach(key => {
      const existing = prev[key]
      if (existing) {
        // allow multiple mutation objects to contain duplicate
        // handlers for the same mutation type
        if (allowDuplicate) {
          if (Array.isArray(existing)) {
            existing.push(obj[key])
          } else {
            prev[key] = [prev[key], obj[key]]
          }
        } else {
          console.warn(`[vuex] Duplicate action: ${ key }`)
        }
      } else {
        prev[key] = obj[key]
      }
    })
    return prev
  }, {})
}
```

注意，这里的调用`mergeObjects`函数时，传入的第二个参数为`true`，因此上述代码可以简化一下：

```js
function mergeObjects (arr, allowDuplicate) {
  return arr.reduce((prev, obj) => {
    Object.keys(obj).forEach(key => {
      const existing = prev[key]
      if (existing) {
        if (Array.isArray(existing)) {
          existing.push(obj[key])
        } else {
          prev[key] = [prev[key], obj[key]]
        }
      } else {
        prev[key] = obj[key]
      }
    })
    return prev
  }, {})
}
```

仔细阅读上述代码，其逻辑还是比较简单的：

 - `arr.reduce`函数负责将数组最终合并成一个对象
 - 合并算法是，针对数组的每一个元素，将其Key-Value赋值给最终合并的对象
 - 遇到重复的key，则使用数组的方式依次存储每一个value

在这个裁剪版本的 shopping-cart 示例中，没有遇到重复 mutation key 重复的情况(其实，我裁剪掉了～～)，在下一个版本中我们在解析这种场景。
