# vuex.get

## 读取数据

在App.vue组件中是通过下述代码获取vuex中的数据的：

```html
      <ul class="todo-list">
        <todo v-for="todo in todos" :todo="todo"></todo>
      </ul>
```

```js
import vuex from '../vuex'

export default {
  data () {
    return {
      todos: vuex.get('todos')
    }
  },
}
```

`vuex.get('todos')`函数的实现代码如下：

```js
import Cursor from './cursor'

export default class Vuex {
  constructor ({
    state = {}
  } = {}) {
    // use a Vue instance to store the state tree
    this._vm = new Vue({
      data: state
    })
  }

  get (path) {
    return new Cursor(this._vm, path)
  }
}
```

那么这个时候，App.vue组件中的`todos`的值就是一个`Cursor`对象的实例，对象初始化的参数有两个：`this._vm`(实际是一个vue实例) 和 `path`(这里的path的值是todos)。

## `Cursor`对象

`Cursor`对象的实现代码如下：

```js
export default class Cursor {
  constructor (vm, path) {
    this.cb = null
    this.vm = vm
    this.path = path
    vm.$watch(path, value => {
      if (this.cb) {
        this.cb.call(null, value)
      }
    })
  }
  get () {
    return this.vm.$get(this.path)
  }
  subscribe (cb) {
    this.cb = cb
  }
}
```

分为以下三个部分：

 - 类构造部分
 - `get`函数实现部分
 - 对外的`subscribe`注册回调函数

这里最关键的两个部分是：`vm.$watch`和`subscribe`函数。因为传进来的`vm`参数实际上是一个vue实例，这里通过`vm.$watch`监听`path`的值是否有改变，如果有改变则通过`subscribe`接口注册的`cb`传递出去。

那么，`subscribe`和`get`函数是在哪里有被调用到呢？实际就在vuex代码中的另外一个文件：mixin.js。

## `init`生命周期钩子(Vue.js 1.x)

在Vue.js 1.x的生命周期钩子中，有一个`init`函数，描述如下：
> 在实例开始初始化时同步调用。此时数据观测、事件和Watcher都尚未初始化。

在vuex的实现代码中，通过`Vue.mixin`在所有vux实例中都增加`init`生命周期钩子。代码如下：

 - index.js

```js
import mixin from './mixin'

Vuex.install = function (_Vue) {
  Vue = _Vue
  Vue.mixin(mixin)
}
```

 - mixin.js

```js
import Cursor from './cursor'

export default {
  init () {
    const dataFn = this.$options.data
    if (dataFn) {
      this.$options.data = () => {
        const raw = dataFn()
        Object.keys(raw).forEach(key => {
          const val = raw[key]
          if (val instanceof Cursor) {
            raw[key] = val.get()
            if (val.cb) {
              throw new Error(
                '[vue-store] A vue-store can only be subscribed to once.'
              )
            }
            val.subscribe(value => {
              this[key] = value
            })
          }
        })
        return raw
      }
    }
  }
}
```

这段代码的主要包含以下逻辑：

 - 先找到vue实例中的`this.$options.data`成员
 - 针对`data`成员中的每一个子成员，判断其值类型是否是`Cursor`类型
 - 对于`Cursor`类型的成员，调用`val.get`先获取当前`state`的值
 - 然后通过`val.subscribe`注册回调函数，当`state`的值发生改变时及时更新

因此，执行完`init`生命周期钩子后，组件中的`todos`的值就由`Cursor`对象实例变为对应的`state`的值，而且每当`state`中`todos`的值发送改变时，就会同步到组件的`data`成员中对应的子成员。而`data`成员的改动自有vue来动态更新页面中显示的内容。
