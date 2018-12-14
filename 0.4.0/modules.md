# modules

## 改动前后使用差异

 - 改动前所有都定义在一级参数上

```js
  it('direct dispatch', () => {
    const vuex = new Vuex({
      state: {
        a: 1
      },
      mutations: {
        [TEST] (state, n) {
          state.a += n
        }
      }
    })
    vuex.dispatch(TEST, 2)
    expect(vuex.state.a).to.equal(3)
  })
```

 - 改动后使用 modules 来定义

```js
  it('modules', function () {
    const store = new Vuex.Store({
      modules: {
        one: {
          state: { a: 2 },
          mutations: {
            [TEST] (state, n) {
              state.a += n
            }
          }
        }
      }
    })
    store.dispatch(TEST, 1)
    expect(store.state.one.a).to.equal(3)
  })
```

## modules 解析和处理

 - 构造函数

```js
  constructor ({
    state = {},
    mutations = {},
    modules = {}
  } = {}) {
    this._rootMutations = this._mutations = mutations

    this._setupModuleState(state, modules)
    this._setupModuleMutations(modules)
  }
```

从构造函数上来看，只处理了 modules 的`state`和`mutation`。

 - _setupModuleState 函数实现

```js
  _setupModuleState (state, modules) {
    const { setPath } = Vue.parsers.path
    Object.keys(modules).forEach(key => {
      setPath(state, key, modules[key].state)
    })
  }
```

把所有的 modules 的`state`都合并到根`state`下

 - _setupModuleMutations 函数实现

```js
  _setupModuleMutations (modules) {
    const { getPath } = Vue.parsers.path
    const allMutations = [this._rootMutations]
    Object.keys(modules).forEach(key => {
      const module = modules[key]
      // bind mutations to sub state tree
      const mutations = {}
      Object.keys(module.mutations).forEach(name => {
        const original = module.mutations[name]
        mutations[name] = (state, ...args) => {
          original(getPath(state, key), ...args)
        }
      })
      allMutations.push(mutations)
    })
    this._mutations = mergeObjects(allMutations)
  }
```

首先把每一个 module 的`mutations`绑定到对应的`state`上，然后再把所有`mutations`合并成一个对象，保证`dispatch`的时候能够触发所有关联的`mutation`。

## setPath

这里的`setPath`函数是 Vue 的一个内部模块暴露出的一个函数，实际意义呢，在后续有一个版本更改中直接改成了`Vue.set`操作：

```js
  _setupModuleState (state, modules) {
    Object.keys(modules).forEach(key => {
      Vue.set(state, key, modules[key].state)
    })
  }
```

这样其实就好理解了，其实就是对象子成员赋值，只是加了一些 Vue 的数据监听和处理。
