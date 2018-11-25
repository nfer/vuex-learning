# 0.0.1

这一版对照的是vuex代码中的 [76ead88](https://github.com/vuejs/vuex/commit/76ead88) 版本的精简代码。

使用 vuex 的代码如下：

```js
import vuex from '../vuex'

const { clearCompleted } = vuex.actions

export default {
  data () {
    return {
      todos: vuex.get('todos')
    }
  },
  methods: {
    clearCompleted
  }
}
```

 - 操作 action 的代码是： `const { clearCompleted } = vuex.actions`
 - 读取数据的代码是： `todos: vuex.get('todos')`

具体代码分析见各自章节。
