# watch

这一版还增加了`watch`接口，测试代码如下：

```js
  it('watch', function (done) {
    const store = new Vuex.Store({
      state: {
        a: 1
      },
      mutations: {
        [TEST]: state => state.a++
      }
    })
    let watchedValueOne, watchedValueTwo
    store.watch(({ a }) => a, val => {
      watchedValueOne = val
    })
    store.watch('a', val => {
      watchedValueTwo = val
    })
    store.dispatch(TEST)
    Vue.nextTick(() => {
      expect(watchedValueOne).to.equal(2)
      expect(watchedValueTwo).to.equal(2)
      done()
    })
  })
```

注意看，这里使用`watch`接口的时候，支持监听整个`state`，也支持通过 key 的形式监听`state`的成员。

## 代码实现

源码实现起来非常简单：

```js
  watch (expOrFn, cb, options) {
    return this._vm.$watch(() => {
      return typeof expOrFn === 'function'
        ? expOrFn(this.state)
        : this._vm.$get(expOrFn)
    }, cb, options)
  }
```

其实就是利用了 vue 的`$watch`监听数据，有改变时通过`callback`通知使用者。
