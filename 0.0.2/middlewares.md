# middlewares

## 定义 action

```js
  dispatch (type, ...payload) {
    const mutation = this._mutations[type]
    if (mutation) {
      mutation(this.state, ...payload)
      this._middlewares.forEach(middleware => {
        middleware({ type, payload }, this.state)
      })
    } else {
      console.warn(`[vuex] Unknown mutation: ${ type }`)
    }
  }

function createAction (action, vuex) {
  return (...payload) => {
    vuex.dispatch(action, ...payload)
  }
}
```

可以看到，在执行action时，会把`{ type, payload }`作为`action`，以及`this.state`传递给每一个middleware函数中进行处理。

## middlewares模块

在 todomvc 中，middlewares模块的代码如下：

```js
import { STORAGE_KEY } from './index'

export default [
  function (action, state) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state.todos))
  }
]
```

即：当每次有 action 改变 state 时，都会将 `state.todos` 数据JSON化后保存在 localStorage 中。

## 读取 localStorage

```js
export const STORAGE_KEY = 'todos-vuejs'
const state = {
  todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
}
```

可以看到，在初始化 vuex 时，通过 localStorage 来初始化 `state.todos` ，这样每次关闭页面后数据并不会丢失，再次打开页面还可以加载上一次的数据。
