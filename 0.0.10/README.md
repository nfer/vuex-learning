# 0.0.10 禁止 mutation 之外修改 state

这一版对照的是vuex代码中的 [4c7e6f6](https://github.com/vuejs/vuex/commit/4c7e6f6) 版本的代码。

所有针对 state 的修改都需要通过 action => mutation 来操作，这样最大的好处就是保证**所有的状态变更都能被调试工具跟踪到**。

## 监听代码

```js
  constructor ({
    development = false
  } = {}) {
    this._dispatching = false
    // add extra warnings in debug mode
    if (development) {
      this._setupMutationCheck()
    }
  }

  _setupMutationCheck () {
    // a hack to get the watcher constructor from older versions of Vue
    // mainly because the public $watch method does not allow sync
    // watchers.
    const unwatch = this._vm.$watch('__vuex__', a => a)
    const Watcher = this._vm._watchers[0].constructor
    unwatch()
    new Watcher(this._vm, '$data', () => {
      if (!this._dispatching) {
        throw new Error(
          '[vuex] Do not mutate vuex state outside mutation handlers.'
        )
      }
    }, { deep: true })
  }
```

注意，这里如果`this._vm, '$data'`的数据有改变时，因为设置了`deep: true`参数，即使是子成员元素改变，都会触发监听函数，那么就会抛出异常信息。

还有就是`sync`参数，这是一个非公开参数，但是在这里是需要的。通过测试，不加这个参数的化，会在程序内部出异常，但是在调用的地方捕获不了这个异常。

## mutation 处理

直接修改`state`的数据会因此报错，那么使用 mutation 操作呢？代码在`dispatch`实现里做了特殊处理：

```js
  dispatch (type, ...payload) {
    this._dispatching = true
    const mutation = this._mutations[type]
    if (mutation) {
      // apply the mutation
      ...
    } else {
      console.warn(`[vuex] Unknown mutation: ${ type }`)
    }
    this._dispatching = false
  }
```

即，在 mutation 前将`this._dispatching`置为`true`， mutation 操作之后再还原。
