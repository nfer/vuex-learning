# 0.5.x

注：后续所有源码解析都是针对于 minor 版本的最后一个 patch 小版本，作为一个阶段的总结。

## createLogger

尤大对于`createLogger`接口是做了好几次的调整，在0.5.x版本中又将其从`vuex`的对象中剥离开来。使用方法如下：

```js
import createLogger from '../../../src/middlewares/logger'

  middlewares: process.env.NODE_ENV !== 'production'
    ? [createLogger()]
    : []
```

在`vuex`的实现中也添加相应的警告：

```js
function createLogger () {
  console.warn(
    '[vuex] Vuex.createLogger has been deprecated.' +
    'Use `import createLogger from \'vuex/logger\' instead.'
  )
}

export default {
  Store,
  install,
  createLogger
}
```

## 增加异常处理

 - vuex 中创建 vue 实例的时候使用静默模式

```js
    // use a Vue instance to store the state tree
    // suppress warnings just in case the user has added
    // some funky global mixins
    const silent = Vue.config.silent
    Vue.config.silent = true
    this._vm = new Vue({
      data: state
    })
    Vue.config.silent = silent
```

 - 处理子模块的 state 的时候添加默认值

```js
  _setupModuleState (state, modules) {
    const { setPath } = Vue.parsers.path
    Object.keys(modules).forEach(key => {
      setPath(state, key, modules[key].state || {})
    })
  }
```

 - 处理子模块的 mutations 的时候进行判空

```js
  _setupModuleMutations (modules) {
    Object.keys(modules).forEach(key => {
      const module = modules[key]
      if (!module || !module.mutations) return
      ... ...
    })
  }
```

## 中间件钩子函数增加 store 参数

```js
m.onMutation({ type, payload }, state, this)
m.onInit(state, this)
```

在源码自带的 src/middlewares/devtool.js 中间件中，有如下的处理：

```js
const hook =
  typeof window !== 'undefined' &&
  window.__VUE_DEVTOOLS_GLOBAL_HOOK__

export default {
  onInit (state, store) {
    if (!hook) return
    hook.emit('vuex:init', store)
    hook.on('vuex:travel-to-state', targetState => {
      const currentState = store._vm._data
      store._dispatching = true
      Object.keys(targetState).forEach(key => {
        currentState[key] = targetState[key]
      })
      store._dispatching = false
    })
  },
  onMutation (mutation, state) {
    if (!hook) return
    hook.emit('vuex:mutation', mutation, state)
  }
}
```

其中`__VUE_DEVTOOLS_GLOBAL_HOOK__`表示的是 vue 的调试工具，如果存在该调试工具，则在 vuex 初始化以及有改动时需要通知到 vue 调试工具：

```js
hook.emit('vuex:init', store)
hook.emit('vuex:mutation', mutation, state)
```

同时，在 vuex 初始化的时候，注册一个回调函数，当从调试工具修改 state 时，需要讲数据同步到 vuex 的 state 中：

```js
    hook.on('vuex:travel-to-state', targetState => {
      const currentState = store._vm._data
      store._dispatching = true
      Object.keys(targetState).forEach(key => {
        currentState[key] = targetState[key]
      })
      store._dispatching = false
    })
```

注意：这里的改动都是在`store._dispatching`标记的处理完成的，不会触发到`_setupMutationCheck`的异常操作捕获逻辑。
