# checkout 操作

## 组件代码

在 Cart.vue 中增加下述代码：

```html
    <p><button :disabled="!cart.added.length" @click="checkout">Checkout</button></p>
    <p v-show="cart.lastCheckout">Checkout {{ cart.lastCheckout }}.</p>
```

```js
const { checkout } = vuex.actions

export default {
  methods: {
    checkout
  }
}
```

## action 代码

```js
import shop from '../api/shop'

export function checkout (products) {
  return (dispatch, state) => {
    const savedCartItems = [...state.cart.added]
    dispatch(types.CHECKOUT_REQUEST)
    shop.buyProducts(
      products,
      () => dispatch(types.CHECKOUT_SUCCESS),
      () => dispatch(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

注意， checkout 和之前讲到的 getAllProducts action 一样，都是也是执行一个异步请求，然后再 dispatch 操作，只不过这里 dispatch 了3个 mutation 。

 - ajax 请求前先`dispatch(types.CHECKOUT_REQUEST)`
 - ajax 成功回调后`dispatch(types.CHECKOUT_SUCCESS)`
 - ajax 失败回调后`dispatch(types.CHECKOUT_FAILURE, savedCartItems)`


