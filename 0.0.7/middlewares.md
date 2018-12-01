# 重构中间件

最开始的 middleware 只是一个简单的函数，比如在最开始的 todomvc 示例中：

```js
import { STORAGE_KEY } from './index'

export default [
  function (action, state) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state.todos))
  }
]
```

其作用非常简单，就是将每次 action 改变后的数据存储在 localStorage 中。读取 localStorage 数据的代码在 vuex 实例化的 index.js 文件中：

```js
export const STORAGE_KEY = 'todos-vuejs'
const state = {
  todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
}
```

这样刷新页面后，仍然可以读取之前的数据。

其实这样的代码是不满足 KISS 原理的，针对 localStorage 的操作分散在不同的模块中。

## 定义 middleware 的状态

对 middleware 进行拆分，定义：

 - `onInit (state)`： middleware 初始化的时候调用
 - `onMutation (mutation, state)`：每次发生 mutation 改变的时候调用

按照这种思路，上述 middleware 的代码可以修改为：

```js
const STORAGE_KEY = 'todos-vuejs'

const localStorageMiddleware = {
  onInit (state) {
    state.todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
  },
  onMutation (mutation, { todos }) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  }
}

export default [
  localStorageMiddleware
]
```

而 vuex 实例化的 index.js 代码也可以做一下简化：

```js
const state = {
  todos: []
}
```

这样所有关于 localStorage 的操作全部封装在该模块中，整个功能和 vuex 主逻辑无强耦合。
