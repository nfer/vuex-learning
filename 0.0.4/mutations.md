# 多 mutation 处理函数

在上一个 shopping-cart 示例中，先抛出一个问题：如果每一个商品有库存数量限制，如何处理？

## vuex 的使用部分修改

最直接的想法就是：在点击‘Add to cart’按钮时，判断库存数量并处理。所以在处理`ADD_TO_CART` action 时，不仅需要操作 cart 模块，也需要操作 products 模块。

修改 products 模块的代码，添加`ADD_TO_CART`处理函数：

```js
import { RECEIVE_PRODUCTS, ADD_TO_CART } from '../action-types'

// mutations
export const productsMutations = {
  [ADD_TO_CART] ({ products }, productId) {
    const product = products.find(p => p.id === productId)
    if (product.inventory > 0) {
      product.inventory--
    }
  }
}
```

这里根据 productId 找到对应的 product ，找到后修改其 inventory 的值。

在 vue 页面中根据  inventory 的值进行判断是否设置按钮的 disabled 状态：

```html
      <button
        :disabled="!p.inventory"
        @click="addToCart(p)">
        Add to cart
      </button>
```

在按钮的点击事件处理函数中再次进行确认：

```js
    addToCart (product) {
      if (product.inventory > 0) {
        addToCart(product.id)
      }
    }
```

## vuex 源码部分修改

虽然在 products 和 cart 模块中分别定义了 mutations ，但是在 vuex 实现中是如何处理的呢？

```js
    // mutations
    this._mutations = Array.isArray(mutations)
      ? mergeObjects(mutations, true)
      : mutations
```

通过合并 mutations ，同一个 action 对应的 mutation 目前是以数组的方式存储了不同模块下的多个 mutations 。

```js
  dispatch (type, ...payload) {
    const mutation = this._mutations[type]
    if (mutation) {
      if (Array.isArray(mutation)) {
        mutation.forEach(m => m(this.state, ...payload))
      } else {
        mutation(this.state, ...payload)
      }
    } else {
      console.warn(`[vuex] Unknown mutation: ${ type }`)
    }
  }
```

在执行具体的 action 时，如果判断 mutation 类型是一个数组类型，则依次调用每一个 mutation ，因此在本示例中，会先将产品加入到购物车中，然后再在商品列表中减去对应的数量。

## 参数对应关系

在组件中使用 action 的时候，只传入了一个擦数：

```js
      if (product.inventory > 0) {
        addToCart(product.id)
      }
```

但在 vuex 中转了一层后，传到模块的 mutation 之后，就多了一个 state 参数：

```js
  dispatch (type, ...payload) {
    const mutation = this._mutations[type]
    if (mutation) {
      if (Array.isArray(mutation)) {
        mutation.forEach(m => m(this.state, ...payload))
      } else {
        mutation(this.state, ...payload)
      }
    }
  }
```

在这个示例中，注意 products 和 cart 模块在处理 ADD_TO_CART action 时，第一个参数用了解构语法：

 - products.js

```js
export const productsMutations = {
  [ADD_TO_CART] ({ products }, productId) {
    const product = products.find(p => p.id === productId)
    ...
  }
}
```

 - cart.js

```js
export const productsMutations = {
  [ADD_TO_CART] ({ cart }, productId) {
    const record = cart.added.find(p => p.id === productId)
    ...
  }
}
```

因为在模块内只关心当前模块的数据，所以使用解构语法直接获取当前关心的成员：`{ cart }`和`{ products }`。
