# getter

这一版对照的是vuex代码中的 [3f080f3](https://github.com/vuejs/vuex/commit/3f080f3) 版本的代码。

在之前的代码中都是直接读取 state ，直接读取的话，最大的问题就是不安全，是违背“最小特权原则”的。因此需要通过 getter 只获取需要的信息。

示例代码：

```js
  it('simple action', function () {
    const store = new Vuex.Store({
      state: {
        a: 1
      },
      getters: {
        getA (state) {
          return state.a
        }
      },
    })
    expect(store.state.a).to.equal(3)
    expect(store.getters.getA()).to.equal(3)
  })
```

其中第一种方法就是直接读取`store.state.a`，而第二种方法就是`getters`方法，是不可以有其他操作的，比如删除`store.state.a`。

## vuex 实现

 - 构造函数的时候传入`getters`参数

```js
  constructor ({
    getters = {}
  } = {}) {
    this.getters = Object.create(null)
    this._setupGetters(getters)
  }
```

 - 实现`_setupGetters`函数

```js
  _setupGetters (getters, hot) {
    this._getters = Object.create(null)
    getters = Array.isArray(getters)
      ? mergeObjects(getters)
      : getters
    Object.keys(getters).forEach(name => {
      this._getters[name] = (...payload) => getters[name](this.state, ...payload)
      if (!this.getters[name]) {
        this.getters[name] = (...args) => this._getters[name](...args)
      }
    })
  }
```

注意，这里有几个细节：

  1. 传入的`getters`可以是一个数组，也可以是一个对象
  1. getter 函数的第一个参数是 state ，后面可以再加自定义参数

 - 添加 hot reload 处理

```js
  hotUpdate ({ actions, mutations, getters } = {}) {
    if (actions) {
      this._setupActions(actions, true)
    }
    if (mutations) {
      this._setupMutations(mutations)
    }
    if (getters) {
      this._setupGetters(getters, true)
    }
  }
```

 注意，处理`getters`和`actions`是有第二个参数的，对应的就是在`_setupGetters`函数的实现中：

```js
  _setupGetters (getters, hot) {
    // delete public getters that are no longer present
    // after a hot reload
    if (hot) validateHotModules(this.getters, getters)
  }
```
