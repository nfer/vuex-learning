# function 类型的 action

在该示例中，对外暴露的 vuex.actions 有以下两个：

 - addToCart： 类型为 string
 - getAllProducts： 类型为 function，代码如下：

```js
export function getAllProducts () {
  return dispatch => {
    shop.getProducts(products => {
      dispatch(types.RECEIVE_PRODUCTS, products)
    })
  }
}
```

vuex 中针对 action 处理如下：

```js
function createAction (action, vuex) {
  if (typeof action === 'string') {}
  else if (typeof action === 'function') {
    // thunk action
    return (...args) => {
      const dispatch = (...args) => {
        vuex.dispatch(...args)
      }
      action(...args)(dispatch)
    }
  }
}
```

这里有几个知识点：

 - getAllProducts 本身类型是一个函数
 - createAction 函数的返回值也是一个函数，并赋值给对应的 action name 

`this.actions[name] = createAction(actions[name], this)`

 - getAllProducts 返回值仍然是一个函数，所以在 createAction 函数中，出现两次括号调用

`action(...args)(dispatch, vuex.stat)`

 - `dispatch` 参数是传递给 getAllProducts 返回值的函数，即下述代码中的`dispatch`

```js
export function getAllProducts () {
  return dispatch => {
    shop.getProducts(products => {
      dispatch(types.RECEIVE_PRODUCTS, products)
    })
  }
}
```

 - `dispatch` 同样是一个函数，使用js闭包语法调用`vuex.dispatch`，函数执行的时候包含 vuex 的上下文

## 总结

从 vuex 的使用作为入口进行跟踪：

 - 注册 actions 时，传入的`getAllProducts`的值是一个函数
 - 注册的函数的返回值也是一个函数
 - 返回值函数中有一个关键的参数`dispatch`

从 vuex 的源码作为入口进行跟踪：

 - `vuex.actions`的类型是一个`Object`
 - `vuex.actions[name]`是通过`createAction`函数生成
 - `vuex.actions[name]`的值是一个函数，即`createAction`函数的返回值是一个函数
 - 在 vue 中使用 action 的时候，都是通过函数调用
 - 注册`createAction`的时候，传入的 action 类型是函数类型
 - 在 vuex 中会包一层处理，把`vuex.dispatch`使用闭包的方式传递给 action 注册函数
 - 最终执行的`dispatch`函数会通过`type`对应到具体的`mutation`实现中
