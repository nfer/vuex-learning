# 热更新

首先我们拷贝一份 counter 的代码，对其中的 vuex 部分按照`actions`、`mutations`模块进行划分，然后和尤大 [08d415c](https://github.com/vuejs/vuex/commit/08d415c) 的版本对比，可以看到只增加了下述代码：

```js
if (module.hot) {
  module.hot.accept(['./actions', './mutations'], () => {
    const newActions = require('./actions').default
    const newMutations = require('./mutations').default
    vuex.hotUpdate({
      actions: newActions,
      mutations: newMutations
    })
  })
}
```

这段代码看上去非常好理解：

 - webpack 会监听`'./actions', './mutations'`文件的变化
 - 文件变动后，重新加载这两个模块
 - 通知 vuex 进行更新

## require().default

上面的代码有一个疑问点：`require('./actions').default`是什么语法？

> export 和 import 的一个重要的限制是，它们必须在其它语句和函数之外使用。不允许出现在`if`语句中，不能有条件的导出和导入。只能在模块顶部使用。

-- 引用于《深入理解ES6》第13章 p319

> 当在node中处理ES6 模块(export default mycomponent)导入的时候，导出的模块格式为

>```js
{
  "default": mycomponent
}
```

> import 语句正确地为你处理了这个问题。 HMR(热更新模块)不在 inline 模式工作的情况下，接口代码不能使用 import ，如果你想避免，使用module.exports而不是export default;

> export 关键字是导出一个对象，对象内存在一个属性(我们要暴露的)，export default 则是 export 语法糖，import 一个export default 暴露出来的模块包含了解构赋值的步骤，所以在node中使用require去请求一个export default的模块需要我们通过.语法去取出对象中的属性(因为require没有解构赋值)

-- 引用于[简书](https://www.jianshu.com/p/0cc58bcbd24c)

## vuex 源码部分

vuex 实现中增加了`hotUpdate`函数：

```js
  hotUpdate ({ actions, mutations } = {}) {
    if (actions) {
      this._setupActions(actions, true)
    }
    if (mutations) {
      this._setupMutations(mutations)
    }
  }
```

同时，在`_setupActions`函数中，增加了一个私有变量`this._actions`：

```js
  _setupActions (actions, hot) {
    // keep the real action functions in an internal object,
    // and expose the public object which are just wrapper
    // functions that point to the real ones. This is so that
    // the reals ones can be hot reloaded.
    this._actions = Object.create(null)
    actions = Array.isArray(actions)
      ? mergeObjects(actions)
      : actions
    Object.keys(actions).forEach(name => {
      this._actions[name] = createAction(actions[name], this)
      if (!this.actions[name]) {
        this.actions[name] = () => this._actions[name]()
      }
    })
    // delete public actions that are no longer present
    // after a hot reload
    if (hot) {
      Object.keys(this.actions).forEach(name => {
        if (!actions[name]) {
          delete this.actions[name]
        }
      })
    }
  }
```

## 总结

经过测试，不做上述改动，示例代码也是可以正常 hot-reload 的，个人猜测是 webpack-dev-server 升级了，有些处理已经不需要手动维护了。

同样，还有几个疑问点：

 - `this._actions`私有变量的存在是否有必要
 - `delete this.actions[name]`的操作不需要专门针对`hot`做判断
 - 所有针对`action`的处理，是不是同样要在`mutation`也有对应的处理
