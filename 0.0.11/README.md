# chat example

页面主结构：

```html
  <div class="chatapp">
    <thread-section></thread-section>
    <message-section></message-section>
  </div>
```

## thread section

主要涉及到以下几处和 vuex 相关：

 - ThreadSection.vue

```js
  computed: {
    threads () {
      return vuex.state.threads
    },
    unreadCount () {
      const threads = vuex.state.threads
      return Object.keys(threads).reduce((count, id) => {
        return threads[id].lastMessage.isRead
          ? count
          : count + 1
      }, 0)
    }
  }
```

 - Thread.vue

```js
  computed: {
    isCurrentThread () {
      return this.thread.id === vuex.state.currentThreadID
    }
  },
  methods: {
    onClick () {
      vuex.actions.switchThread(this.thread.id)
    }
  }
```

## message section

主要涉及到以下几处和 vuex 相关：

 - MessageSection.vue

```js
  computed: {
    thread () {
      const id = vuex.state.currentThreadID
      return id
        ? vuex.state.threads[id]
        : {}
    },
    messages () {
      return this.thread.messages &&
        this.thread.messages.map(id => vuex.state.messages[id])
    }
  },
  watch: {
    'thread.lastMessage': function () {
      this.$nextTick(() => {
        const ul = this.$els.list
        ul.scrollTop = ul.scrollHeight
      })
    }
  },
  methods: {
    sendMessage (e) {
      const text = e.target.value
      if (text.trim()) {
        vuex.actions.sendMessage(text, this.thread)
        e.target.value = ''
      }
    }
  }
```

## mutation

chat 示例中共有三个 mutation-type ：

 - SWITCH_THREAD
 - RECEIVE_ALL
 - RECEIVE_MESSAGE

最简单的就是`SWITCH_THREAD`，实现如下：

```js
function setCurrentThread (state, id) {
  state.currentThreadID = id
  // mark thread as read
  state.threads[id].lastMessage.isRead = true
}
```

`RECEIVE_MESSAGE`的实现也很简单，把收到的`message`保存到 state 中即可：

```js
function addMessage (state, message) {
  // add a `isRead` field before adding the message
  message.isRead = message.threadID === state.currentThreadID
  // add it to the thread it belongs to
  const thread = state.threads[message.threadID]
  if (!thread.messages.some(id => id === message.id)) {
    thread.messages.push(message.id)
    thread.lastMessage = message
  }
  // add it to the messages map
  set(state.messages, message.id, message)
}
```

注意，这里的`set`操作展开了来是`Vue.set(state.messages, message.id, message)`。

`RECEIVE_ALL`的实现略微有点复杂：

```js
  [types.RECEIVE_ALL] (state, messages) {
    let latestMessage
    messages.forEach(message => {
      // create new thread if the thread doesn't exist
      if (!state.threads[message.threadID]) {
        createThread(state, message.threadID, message.threadName)
      }
      // mark the latest message
      if (!latestMessage || message.timestamp > latestMessage.timestamp) {
        latestMessage = message
      }
      // add message
      addMessage(state, message)
    })
    // set initial thread to the one with the latest message
    setCurrentThread(state, latestMessage.threadID)
  },
```

代码注释基本描述清晰了，这里不再赘述。
