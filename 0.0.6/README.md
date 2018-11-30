# 0.0.6

这一版对照的是vuex代码中的 [4411384](https://github.com/vuejs/vuex/commit/4411384) 和 [342e6fc](https://github.com/vuejs/vuex/commit/342e6fc) 两个版本结合的代码。

从这一版开始，尤大完全抛弃了之前的 mixin 和 Cursor ，而是直接通过 vue 的`computed`属性来获取`vuex.state`。
