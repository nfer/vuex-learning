# 测试用例

这一版代码从以下几个维度对 vuex 进行了测试：

  - direct dispatch
  - simple action
  - async action
  - array option syntax
  - hot reload
  - middleware
  - middleware with snapshot

## direct dispatch

测试步骤：

 1. 实例化一个 vuex
 1. 直接调用`vuex.dispatch`
 1. 检查`vuex.state`结果的值

测试代码：

```js
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
```

## simple action

测试步骤：

 1. 实例化一个 vuex
 1. 调用`vuex.actions.xxx`触发改变
 1. 检查`vuex.state`结果的值

测试代码：

```js
    const vuex = new Vuex({
      ...
      actions: {
        test: TEST
      },
      ...
    })
    vuex.actions.test(2)
    expect(vuex.state.a).to.equal(3)
```

## async action

测试步骤：

 1. 实例化 vuex ，传入一个异步 action
 1. 调用`vuex.actions.xxx`触发改变
 1. 延时检查`vuex.state`结果的值

测试代码：

```js
    const vuex = new Vuex({
      ...
      actions: {
        test: (n) => (dispatch, state) => {
          setTimeout(() => {
            dispatch(TEST, n)
          }, 10)
        }
      },
      ...
    })
    vuex.actions.test(2)
    setTimeout(() => {
      expect(vuex.state.a).to.equal(3)
      done()
    }, 10)
```

## array option syntax

测试步骤：

 1. 实例化 vuex ，传入 actions 和 mutations 都是数组类型
 1. 调用`vuex.actions.test`触发改变，会涉及到多个 mutation
 1. 检查`vuex.state`结果的值
 1. 调用`vuex.actions.test2`触发改变
 1. 再次检查`vuex.state`结果的值

测试代码：

```js
    const vuex = new Vuex({
      state: {
        a: 1,
        b: 1,
        c: 1
      },
      actions: [{ test: TEST }, { test2: TEST2 }],
      mutations: [
        {
          [TEST] (state, n) {
            state.a += n
          }
        },
        // allow multiple handlers for the same mutation type
        {
          [TEST] (state, n) {
            state.b += n
          },
          [TEST2] (state, n) {
            state.c += n
          }
        }
      ]
    })
    vuex.actions.test(2)
    expect(vuex.state.a).to.equal(3)
    expect(vuex.state.b).to.equal(3)
    expect(vuex.state.c).to.equal(1)
    vuex.actions.test2(2)
    expect(vuex.state.c).to.equal(3)
```

## hot reload

测试步骤：

 1. 实例化一个 vuex
 1. 调用`vuex.actions.xxx`触发改变
 1. 检查`vuex.state`结果的值
 1. 热更新 actions 和 mutations
 1. 再次触发 action 并检查`vuex.state`结果


测试代码：

```js
    const vuex = new Vuex({...})
    const test = vuex.actions.test
    test(2)
    expect(vuex.state.a).to.equal(3)
    vuex.hotUpdate({
      actions: {
        test: n => dispatch => dispatch(TEST, n + 1)
      },
      mutations: {
        [TEST] (state, n) {
          state.a = n
        }
      }
    })
    test(999)
    expect(vuex.state.a).to.equal(1000)
```

## middleware

测试步骤：

 1. 实例化 vuex ，并注册 middlewares
 1. 触发 action
 1. 检查`vuex.state`结果以及 middlewares 中存储的结果

测试代码：

```js
    const vuex = new Vuex({
      ...
      middlewares: [
        {
          onInit (state) {
            initState = state
          },
          onMutation (mut, state) {
            expect(state).to.equal(vuex.state)
            mutations.push(mut)
          }
        }
      ]
    })
    expect(initState).to.equal(vuex.state)
    vuex.actions.test(2)
    expect(mutations.length).to.equal(1)
    expect(mutations[0].type).to.equal(TEST)
    expect(mutations[0].payload[0]).to.equal(2)
```

## middleware with snapshot

测试步骤：

 1. 实例化 vuex ，并注册带有`snapshot`属性的 middlewares
 1. 触发 action
 1. 检查`vuex.state`结果以及 middlewares 中存储的结果

测试代码：

```js
    const vuex = new Vuex({
      ...
      middlewares: [
        {
          snapshot: true,
          onInit (state) {
            initState = state
          },
          onMutation (mutation, nextState, prevState) {
            mutations.push({
              mutation,
              nextState,
              prevState
            })
          }
        }
      ]
    })
    expect(initState).not.to.equal(vuex.state)
    expect(initState.a).to.equal(1)
    vuex.actions.test(2)
    expect(mutations.length).to.equal(1)
    expect(mutations[0].mutation.type).to.equal(TEST)
    expect(mutations[0].mutation.payload[0]).to.equal(2)
    expect(mutations[0].nextState).not.to.equal(vuex.state)
    expect(mutations[0].prevState.a).to.equal(1)
    expect(mutations[0].nextState.a).to.equal(3)
```
