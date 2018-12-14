# injecting

这一版还增加了在实例化 vue 的时候，注入`state`和`actions`到 vue 的实例中，测试代码如下：

```js
  it('injecting state and action to components', function () {
    const store = new Vuex.Store({
      state: {
        a: 1
      },
      mutations: {
        [TEST] (state, n) {
          state.a += n
        }
      }
    })
    const vm = new Vue({
      store,
      vuex: {
        state: {
          b: state => state.a
        },
        actions: {
          test: ({ dispatch }, n) => dispatch(TEST, n)
        }
      }
    })
    vm.test(2)
    expect(vm.b).to.equal(3)
    expect(store.state.a).to.equal(3)
  })
```

注意，这里我们既可以通过`store.state.a`来读取，也可以用`vm.b`来读取真正的`state`的数据，而且可以把`actions`的函数注入到 vue 的实例中。

## override

所有的操作是在挂载 vuex 的时候，进行处理：

 - index.js

```js
import override from './override'

function install (_Vue) {
  Vue = _Vue
  override(Vue)
}
```

 - override.js

```js
// export install function
export default function (Vue) {
  const _init = Vue.prototype._init
  Vue.prototype._init = function (options) {
    options = options || {}
    // vuex option handling
    const vuex = options.vuex || componentOptions.vuex
    if (vuex) {
      const { state, actions } = vuex
      // state
      if (state) {
        options.computed = options.computed || {}
        Object.keys(state).forEach(key => {
          options.computed[key] = function vuexBoundGetter () {
            return state[key].call(this, this.$store.state)
          }
        })
      }
      // actions
      if (actions) {
        options.methods = options.methods || {}
        Object.keys(actions).forEach(key => {
          options.methods[key] = function vuexBoundAction (...args) {
            return actions[key].call(this, this.$store, ...args)
          }
        })
      }
    }
    _init.call(this, options)
  }
}
```

从源码上可以看到，分别把`state`的数据绑定到了`computed`下，把`actions`的函数绑定到了`methods`下。
