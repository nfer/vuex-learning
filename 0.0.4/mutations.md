# 多 mutation 处理函数

在上一个 shopping-cart 示例中，先抛出一个问题：如果每一个商品有库存数量限制，如何处理？

最直接的想法就是：在点击‘Add to cart’按钮时，判断库存数量并处理。所以在处理`ADD_TO_CART` action 时，不仅需要操作 cart 模块，也需要操作 products 模块。

修改 products 模块的代码，也添加`ADD_TO_CART`处理函数：

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
