# actions

## 注册actions

 - 创建actions.js文件，其内容是Key-Value格式的结构体：

```js
export default {
  addTodo: 'ADD_TODO',
  deleteTodo: 'DELETE_TODO',
  toggleTodo: 'TOGGLE_TODO',
  editTodo: 'EDIT_TODO',
  toggleAll: 'TOGGLE_ALL',
  clearCompleted: 'CLEAR_COMPLETED'
}
```

 - 在创建vuex的文件index.js中，引入actions.js，并注册到vuex中：

```js
import actions from './actions'

const vuex = new Vuex({
  ...
  actions,
  ...
})
```

 - 在vuex的实现中，创建实际的action处理函数

```js
export default class Vuex {
  constructor ({
    ...
    actions = {},
    ...
  } = {}) {
    ...
    // create actions
    this.actions = Object.create(null)
    Object.keys(actions).forEach(name => {
      this.actions[name] = createAction(actions[name], this)
    })
    ...
  }
}

function createAction (action, vuex) {
  return (...payload) => {
    vuex.dispatch(action, ...payload)
  }
}
```

注意这个时候，Vuex对象暴露出一个成员变量`this.actions`，变量的类型是一个对象，通过`name`来映射真正的action处理函数。

## 使用action

下面以`clearCompleted`为例说明一下如何使用action：

```html
      <button class="clear-completed"
        v-show="todos.length > remaining"
        @click="clearCompleted">
        Clear completed
      </button>
```

```js
import vuex from '../vuex'
const { clearCompleted } = vuex.actions

export default {
  methods: {
    clearCompleted
  }
}
```

在创建vuex的时候，会`export`出生成的`vuex`实例，其中`vuex.actions`就是实例暴露出来的一个成员。

`const { clearCompleted } = vuex.actions`会解构出`clearCompleted`对应的处理函数，如果需要在vue的template中使用，还需要在vue组件实例的`methods`中进行一下映射：

```js
export default {
  methods: {
    clearCompleted: clearCompleted // 可以直接简写为clearCompleted
  }
}
```

这个时候再详细看一下注册时候调用的`dispatch`函数实现：

```js
export default class Vuex {
  constructor ({
    state = {},
    mutations = {}
  } = {}) {
    // use a Vue instance to store the state tree
    this._vm = new Vue({
      data: state
    })

    // mutations
    this._mutations = mutations
  }

  dispatch (type, ...payload) {
    const mutation = this._mutations[type]
    if (mutation) {
      mutation(this.state, ...payload)
    } else {
      console.warn(`[vuex] Unknown mutation: ${ type }`)
    }
  }

  get state () {
    return this._vm._data
  }
}
```

在定义actions时传的是`{name: type}`的组合，在`dispatch`的时候，则通过`type`再去`mutations`中查找对应的函数：

```js
export default {
  ADD_TODO (state, text) {
    state.todos.unshift({
      text: text,
      done: false
    })
  },

  DELETE_TODO (state, todo) {
    state.todos.$remove(todo)
  },

  TOGGLE_TODO (state, todo) {
    todo.done = !todo.done
  },

  EDIT_TODO (state, todo, text) {
    todo.text = text
  },

  TOGGLE_ALL (state, done) {
    state.todos.forEach((todo) => {
      todo.done = done
    })
  },

  CLEAR_COMPLETED (state) {
    state.todos = state.todos.filter(todo => !todo.done)
  }
}
```

## 总结

为什么绕了整整一个大圈，定义`actions`、`mutations`，然后再通过Vuex中转一次来进行调用？

最核心的部分是就是注册actions时候的`createAction`函数：

```js
function createAction (action, vuex) {
  return (...payload) => {
    vuex.dispatch(action, ...payload)
  }
}
```

这里通过js的闭包语法，把创建的vuex实例(对象中包含了`state`存储对象)包裹在函数的调用上下文中，这样在使用action的时候就不需要关心`state`，只需要具体的action参数即可。
