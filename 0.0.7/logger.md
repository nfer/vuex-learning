# logger 中间件

为了调试方便，需要将每一个 mutation 的状态输出出来，因此增加了 logger 中间件。调试的时候需要以下信息：

 - mutation 名称
 - mutation 改变后的 state 状态
 - mutation 改变前的 state 状态

因此对应到接口参数上代码如下：

```js
    onMutation (mutation, nextState, prevState) {
    }
```

`onMutation`函数的实现，简单了来讲直接`console.log`即可，尤大实现的时候参考了`fcomb/redux-logger`的代码，做了格式化输出，具体的格式这里展开，自己看源码即可。

## prevState

相对于 devtool.js 文件定义的基准中间件， logger.js 文件的接口中多了一个`prevState`参数。

 - 创建 vuex 时(构造函数)，判断是否有中间件需要缓存 state ，如果需要则深拷贝一份：

```js
  constructor ({
    state = {},
    middlewares = []
  } = {}) {
    this._needSnapshots = middlewares.some(m => m.snapshot)
    this._prevSnapshot = this._needSnapshots
      ? deepClone(state)
      : null
  }
```

 - 每次 mutation 改变时，先获取上一次缓存的 state ，并传给需要的中间件

```js
  dispatch (type, ...payload) {
    const mutation = this._mutations[type]
    const prevSnapshot = this._prevSnapshot
    const state = this.state
    
      this._middlewares.forEach(m => {
        if (m.snapshot) {
          m.onMutation({ type, payload }, state, prevSnapshot)
        } else {
          m.onMutation({ type, payload }, state)
        }
      })
  }
```

 - mutation 改变后，同样需要缓存当前 state ，以便在下一次输出的时候使用和对比

```js
  dispatch (type, ...payload) {
    const mutation = this._mutations[type]
    const state = this.state

      // apply the mutation
      if (Array.isArray(mutation)) {
        mutation.forEach(m => m(state, ...payload))
      } else {
        mutation(state, ...payload)
      }
      // invoke middlewares
      if (this._needSnapshots) {
        this._prevSnapshot = deepClone(state)
      }
  }
```

通过上述修改可以看到，可以正常输出`prevState`。

## 其他参数深拷贝

除了`prevState`需要缓存，在`console.log`的时候也需要缓存其他对象。因为在 js 中，参数类型为对象时是传引用而不是传值，因此如果直接传`state`到`m.onMutation`中，实际情况就会出现多次输出的内容是一样的，因此也要进行对应的深拷贝。

```js
  dispatch (type, ...payload) {
    const prevSnapshot = this._prevSnapshot
    const state = this.state
    let snapshot, clonedPayload

      // invoke middlewares
      if (this._needSnapshots) {
        snapshot = this._prevSnapshot = deepClone(state)
        clonedPayload = deepClone(payload)
      }
      this._middlewares.forEach(m => {
        if (m.snapshot) {
          m.onMutation({ type, payload: clonedPayload }, snapshot, prevSnapshot)
        } else {
          m.onMutation({ type, payload }, state)
        }
      })
  }
```

这里对`payload`和`state`都做了深拷贝，然后再进行传参。

## 调整 onInit 顺序

经过测试，尤大这一版代码中有一个小问题，就是第一次发生 mutation 的时候，获取的`prevState`的值是空的。将`onInit`钩子执行的顺序调整到缓存首次`state`的操作之前：

```js
    // call init hooks
    this._middlewares.forEach(m => {
      if (m.onInit) {
        m.onInit(state)
      }
    })
    this._needSnapshots = middlewares.some(m => m.snapshot)
    this._prevSnapshot = this._needSnapshots
      ? deepClone(state)
      : null
```

这样获取的`this._prevSnapshot`就是经过组件处理后的`state`状态，比如在 todomvc 中， localStorage 组件会修改 state 的值。
