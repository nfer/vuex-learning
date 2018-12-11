# bind dispatch to self

这一版对照的是vuex代码中的 [cff54fc8](https://github.com/vuejs/vuex/commit/cff54fc8) 版本的代码。

## 用法上的差别

这一版代码中最大的改动就是 action 类型为函数时的处理，先看一下前后使用的对比：

 - 改动前：

```js
      actions: {
        test: (n) => (dispatch, state) => {
          setTimeout(() => {
            dispatch(TEST, n)
          }, state.timeout)
        }
      },
```

 - 改动后：

```js
      actions: {
        test: ({ dispatch, state }, n) => {
          setTimeout(() => {
            dispatch(TEST, n)
          }, state.timeout)
        }
      },
```

之前的语法中，`test`本身是一个函数，函数内部还返回一个函数，换种写法就是：

```js
        test: function(n) {
          return function(dispatch, state) {
            setTimeout(() => {
              dispatch(TEST, n)
            }, state.timeout)
          }
        }
```

而改动后，`test`本身是一个函数，函数内部就是执行体，没有返回任何函数，换种写法就是：

```js
        test: function({ dispatch, state }, n) {
          setTimeout(() => {
            dispatch(TEST, n)
          }, state.timeout)
        }
```

## 内部实现上的差别

改动前后，针对 action 类型为函数的处理也是不同的：

 - 改动前

```js
export function createAction (action, store) {
  else if (typeof action === 'function') {
    // thunk action
    return (...args) => {
      const dispatch = (...args) => {
        store.dispatch(...args)
      }
      action(...args)(dispatch, store.state)
    }
  }
}
```

 - 改动后

```js
export function createAction (action, store) {
  else if (typeof action === 'function') {
    // normal action
    return (...payload) => action(store, ...payload)
  }
}
```

可以看到：

 - 改动前需要通过`action(...args)(dispatch, store.state)`，两层`()()`来执行，改动后只需要一层`()`
 - 改动前是传递了三个参数`...args`、`dispatch`和`store.state`，改动后，只需要两个参数`store`和`...payload`

注意，在定义 action 的时候，通过解构处理，取得了`store`的成员变量`dispatch`和`state`，所以在 vuex 的实现中做如下修改：

```js
    // bind dispatch to self
    const dispatch = this.dispatch
    this.dispatch = (...args) => {
      dispatch.apply(this, args)
    }
```

## 总结

经过修改后，最大的好处就是，在定义 action 时更直观也更简便。

比如`dispatch`参数，现在就很清晰，是从第一参数解构得来的，而之前的写法则是完全不清晰`dispatch`函数的定义所在。
