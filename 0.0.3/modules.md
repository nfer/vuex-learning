# vuex 模块

## state 拆分

当 vuex 业务复杂后，所有的 state 都写在 index.js 文件中会极大的增加代码复杂度，这个时候就需要进行代码拆分。

在 shopping-cart 示例中，就做了 cart 和 products 拆分：

```js
import { cartInitialState } from './stores/cart'
import { productsInitialState } from './stores/products'

export default new Vuex({
  state: {
    cart: cartInitialState,
    products: productsInitialState
  }
})
```

这里， 对外暴露的 state 有两部分组成： cart 和 products ，其数据分别来自于 './stores/cart' 和 './stores/products' 模块：

 - cart.js

```js
// initial state
export const cartInitialState = {
  added: [],
}
```

 - products.js

```js
// initial state
export const productsInitialState = []
```

## actions

虽然 state 做了拆分，但是对外暴露的 actions 仍然还是定义在一个文件中：

```js
import shop from '../api/shop'
import * as types from './action-types'

export const addToCart = types.ADD_TO_CART

export function getAllProducts () {
  return dispatch => {
    shop.getProducts(products => {
      dispatch(types.RECEIVE_PRODUCTS, products)
    })
  }
}
```

注意，这里的 getAllProducts action 的类型已经是 function 了，和之前 todomvc 示例中的 string action是不同的，稍后的章节中会详细展开。

## mutations 拆分

action 是暴露出来的接口，但是具体的 action 对应的操作是和具体的模块相关，因此这里也把 mutations 拆分到了各自的 module 中：

```js
import { cartMutations } from './stores/cart'
import { productsMutations } from './stores/products'

export default new Vuex({
  mutations: [cartMutations, productsMutations]
})
```

注意，相对于 todomvc 示例中，这里的 mutations 类型已经是一个数组，而不是一个 Key-Value 的对象格式，因此在 vuex 中需要进行对应的处理，稍后的章节中会详细展开。

每个模块的 mutations 自己实现：

 - cart.js

```js
import { ADD_TO_CART } from '../action-types'

// mutations
export const cartMutations = {
  [ADD_TO_CART] ({ cart }, productId) {
    const record = cart.added.find(p => p.id === productId)
    if (!record) {
      cart.added.push({
        id: productId,
        quantity: 1
      })
    } else {
      record.quantity++
    }
  }
}
```

 - products.js

```js
import { RECEIVE_PRODUCTS } from '../action-types'

// mutations
export const productsMutations = {
  [RECEIVE_PRODUCTS] (state, products) {
    state.products = products
  }
}
```
