# injection

这一版对照的是vuex代码中的 [adaa2c3](https://github.com/vuejs/vuex/commit/adaa2c3) 版本的代码。

## vue 全局注册 $store

在通过`Vue.use(Vuex)`安装 vuex 时，会调用`install`方法来做一些初始化的操作：

```js
import Vuex from '../../src'

Vue.use(Vuex)
```

vuex 的`install`实现如下：

```js
export function install (_Vue) {
  Vue = _Vue
  const _init = Vue.prototype._init
  Vue.prototype._init = function (options) {
    options = options || {}
    if (options.store) {
      this.$store = options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
    _init.call(this, options)
  }
}
```

## 使用 $store

 - 创建 vue 的时候传入 vuex 参数

```js
import Vue from 'vue'
import store from './store'
import App from './components/App.vue'

new Vue({
  store, // inject store to all children
  el: 'body',
  components: { App }
})
```

 - 使用`this.$store`来操作 vuex

```js
  computed: {
    todos () {
      return this.$store.state.todos
    }
  },
  methods: {
    addTodo (e) {
      var text = e.target.value
      if (text.trim()) {
        this.$store.actions.addTodo(text)
      }
      e.target.value = ''
    }
  }
```