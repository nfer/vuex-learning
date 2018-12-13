# 0.4.0

## 接口改变

 - 之前的使用方式

```js
  it('simple action', function () {
    const store = new Vuex.Store({
      state: {},
      actions: {},
      getters: {},
      mutations: {}
    })
    store.actions.test(2)
    expect(store.state.a).to.equal(3)
    expect(store.getters.getA()).to.equal(3)
  })
```

Vuex.Store 作为一个单独的模块，独立创建和使用。

 - 新接口使用方式

```js
  it('injecting state and action to components', function () {
    const store = new Vuex.Store({
      state: {},
      mutations: {}
    })
    const vm = new Vue({
      store,
      vuex: {
        state: {
          a: state => state.a
        },
        actions: {
          test: ({ dispatch }, n) => dispatch(TEST, n)
        }
      }
    })
    vm.test(2)
    expect(vm.a).to.equal(3)
    expect(store.state.a).to.equal(3)
  })
```

注意，新接口下，创建 Vuex.Store 实例时，只传入`state`、`mutation`参数。然后把整个 store 作为一个参数传递给 Vue 的实例，然后再传入 vuex 模块(包含 state 和 actions )。

注意， getters 整个被拿掉了，又回到了之前裸读取的情况(`store.state.a`)，囧～～

## 版本变化

 - injecting state and action to components
 - modules
 - watch

middleware 部分和 strict mode 检验几乎没有变化，hot reload 部分改动比较大，但是在之前的版本分析中就发现和 webpack 版本脱节太大，暂不分析，留待最新版本再专门处理。
