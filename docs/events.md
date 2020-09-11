# Vue 事件的高级使用方法

## 事件方法

在Vue中提供了4中事件监听方法，分别是：

- $on(event: string | Array<String>, fn)
- $emit(event: string)
- $once(event: string, fn)
- $off(event?: string|Array<string>, fn?)

### 1. $on

> 监听当前实例上的自定义事件。事件可以由 vm.$emit 触发。回调函数会接收所有传入事件触发函数的额外参数。

从上述传参可以看出。第一个参数可以传递一个字符串，或者一个数组，如果传递的是数组，则会对数组中的每一个事件进行监听，只要有一个触发，就会执行后面的回调函数

```js
mounted () {
  this.$on('event1', () => {
    console.log('ok');
  });

  this.$on(['event1', 'event2', 'event3'], () => {
    console.log('ok');
  });

  setTimeout(() => {
    this.$emit('event1');
  }, 1000);
}
```

当调用 `$emit` 之后，两个 `$on` 都会被触发

### 2. $emit

> 触发当前实例上的事件。附加参数都会传给监听器回调。

此处的用法非常简单，可以和 `$on` 配合使用，也可以和组件中的自定义事件配合使用

```js
Vue.component('welcome-button', {
  template: `
    <button v-on:click="$emit('welcome')">
      Click me to be welcomed
    </button>
  `
});

<div id="emit-example-simple">
  <welcome-button v-on:welcome="sayHi"></welcome-button>
</div>

methods: {
  sayHi: function () {
    alert('Hi!')
  }
}
```

### 3. $once

> 监听一个自定义事件，但是只触发一次。一旦触发之后，监听器就会被移除。

### 4. $off

> 移除自定义事件监听器

- 如果没有提供参数，则移除所有的事件监听器；
- 如果只提供了事件，则移除该事件所有的监听器；
- 如果同时提供了事件与回调，则只移除这个回调的监听器。


## 高级用法

### 1. 使用$on $emit 实现跨组件通信

有时两个组件隔的比较远的情况下，可以通过这两个组件的共同父元素，或者跟节点来派发和监听事件，达到通信的效果。

```js
// components A
<div>
  <button @click="clickHandler">click me</button>
</div>

export default {
  name: 'componentsA',
  method: {
    clickHandler() {
      this.$root.$emit('my-events');
    }
  }
}

// components B
export default {
  name: 'componentsB',
  mounted () {
    this.$root.$on('my-events', () => {
      console.log('ok');
    });
  }
}
// 这里的$root可以换成共同的父元素
```

### 2. hookEvents

父组件可以直接通过自定义事件的形式监听子组件中声明周期钩子的变化，固定写法 `<componentsA @hook:update="func" />`, 目的是当使用了第三方组件，还想要知道里面的生命周期触发了。可以使用此方法监听

```js
// child Events
export default {
  name: 'Events2',
  data () {
    return {
      counter: 0
    };
  },
  mounted () {
    setTimeout(() => {
      this.counter += 1;
    }, 2000);
  },
  updated () {
    console.log('update');
  }
};

// parent components
<Event2 @hook:updated="updateCounter"/>

export default {
  components: {
    Events2
  },
  methods: {
    updateCounter () {
      console.log('hooks ok');
    }
  }
}
```

## 源码分析

1. 定义事件源码位置：`vue/src/core/instance/events.js`

```js
export function eventsMixin (Vue: Class<Component>) {
  const hookRE = /^hook:/ // hookEvent
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component { // 除了可以使用字符串，还可以监听一个数组，['event1', 'event2']
    const vm: Component = this
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$on(event[i], fn)
      }
    } else {
      // 把事件名称和回调函数存入vm._events中
      (vm._events[event] || (vm._events[event] = [])).push(fn) // 一个事件可以对应多个回调函数
      // optimize hook:event cost by using a boolean flag marked at registration
      // instead of a hash lookup
      if (hookRE.test(event)) { // 监听生命周期的钩子事件
        vm._hasHookEvent = true // 只要有人监听这个事件。在callHooks中就是会派发一个事件
      }
    }
    return vm
  }

  Vue.prototype.$once = function (event: string, fn: Function): Component {
    const vm: Component = this
    function on () {
      vm.$off(event, on) // 执行一次回调函数后就立刻结束
      fn.apply(vm, arguments)
    }
    on.fn = fn
    vm.$on(event, on)
    return vm
  }

  Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {
    const vm: Component = this
    // all
    if (!arguments.length) { // 无参数，清除所有的事件监听
      vm._events = Object.create(null)
      return vm
    }
    // array of events
    if (Array.isArray(event)) { // 传入的数组，把相关的事件移除
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$off(event[i], fn)
      }
      return vm
    }
    // specific event 解除特定事件
    const cbs = vm._events[event]
    if (!cbs) {
      return vm
    }
    if (!fn) { // 如果用户没指定fn参数，相关的所有回调都清除
      vm._events[event] = null
      return vm
    }
    // specific handler // 如果用户指定fn参数，只移除对应的回调
    let cb
    let i = cbs.length
    while (i--) {
      cb = cbs[i]
      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    }
    return vm
  }

  Vue.prototype.$emit = function (event: string): Component { // 事件派发
    const vm: Component = this
    let cbs = vm._events[event]
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs
      const args = toArray(arguments, 1)
      const info = `event handler for "${event}"`
      for (let i = 0, l = cbs.length; i < l; i++) {
        invokeWithErrorHandling(cbs[i], vm, args, vm, info)
      }
    }
    return vm
  }
}
```

2. 派发hookEvents位置：`vue/src/core/instance/lifecycle.js`

```js
export function callHook (vm: Component, hook: string) {
  if (vm._hasHookEvent) { // 如果标记了钩子事件，则额外的派发一个自定义的事件出去
    vm.$emit('hook:' + hook) // 比如：@hook:update="xxx"
  }
}
```

