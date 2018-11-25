# 0.0.2

这一版对照的是vuex代码中的 [76ead88](https://github.com/vuejs/vuex/commit/76ead88) 版本的代码。

在上一章中，我们针对 76ead88 版本的代码做了最小精简，只关注其中的 actions 和 get 部分，这一章我们补齐 76ead88 版本剩余的代码：

```js
import middlewares from './middlewares'

export default new Vuex({
  middlewares
})
```

就是上一章中没有提到的`middlewares`。
