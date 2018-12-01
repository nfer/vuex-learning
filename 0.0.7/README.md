# 0.0.7

这一版对照的是vuex代码中的 [cab36ed](https://github.com/vuejs/vuex/commit/cab36ed) 版本简化后的代码。

到这一版代码，尤大对 vuex 的`middlewares`做了较大改动，主要包括以下几个方面：

 - 针对 middlewares ，定义了`onInit`和`onMutation`两种状态
 - 增加`logger`中间件用于调试
