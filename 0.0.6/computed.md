# 去掉 mixin 和 Cursor

在上一个版本中，通过`todos: vuex.get('todos')`来获取 state 中的数据，其中`vuex.get()`最终会展开为监听回调函数并注册给`vm.$watch()`，从而可以实时更新 data 中的`todos`成员的值。

在这一个版本中，直接通过 computed 语法直接获取 vuex.state 中的值：

```js
import vuex from '../vuex'

export default {
  computed: {
    todos () {
      return vuex.state.todos
    }
  },
}
```

`vuex.state`是通过 vuex 的 getter 函数读取：

```js
export default class Vuex {
  constructor ({
    state = {}
  } = {}) {
    // use a Vue instance to store the state tree
    this._vm = new Vue({
      data: state
    })
  }

  get state () {
    return this._vm._data
  }
}
```

## 疑问1: 使用 computed 还是 methods ？

区别一：computed 是属性，而 methods 是方法

因此，如果改成下述代码：

```js
export default {
  methods: {
    todos () {
      return vuex.state.todos
    }
  },
}
```

那么所有使用到`this.todos`的地方都需要改成`this.todos()`。

区别二：computed 会有缓存，而 methods 不会

我们修改`vuex.state`的 getter 函数，添加一个调试信息：

```js
  get state () {
    console.error('get state', JSON.stringify(this._vm._data));
    return this._vm._data
  }
```

可以非常明显的看到，如果数据不发生改变，使用 computed 时，只有一次`vuex.state.todos`读取，而使用 methods 时则每一次使用`this.todos()`都会发生`vuex.state.todos`读取。

## 疑问2: 使用 computed 属性还是 data 属性？

区别：computed 属性是计算属性，而 data 属性只是一次性的赋值

同样通过添加调试信息，可以看到使用 data 属性时只有组件初始化的时候读取了`vuex.state.todos`的值，后续通过 action 修改 state 中的数据，都不会同步更新到`this.todos`。

虽然，对于有些页面操作，看上去使用 computed 属性和 data 属性没有什么差别，比如在 todomvc 示例中，添加 todo ， 修改 todo 的状态，表现都一致。但是对于另外那些只有 action 没有页面数据的操作，二者表现就会有差异。最典型的例子同样是在 todomvc 示例中，操作步骤如下：

 1. 创建几个 todo
 1. 选择其中的 todo ，切换为 Completed 状态
 1. 点击 clearCompleted 按钮，选择清空所有已完成的 todo

这个时候，使用 computed 属性还是 data 属性，表现就出现了不同。使用 computed 属性时，组件的`this.todos`能够及时和`vuex.state.todos`的值进行同步，页面显示正常；而使用 data 属性时，由于只有 action 操作，组件内的`this.todos`并没有改变，和`vuex.state.todos`的值产生了差异。
