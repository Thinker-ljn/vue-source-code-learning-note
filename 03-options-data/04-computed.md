# computed 计算属性

> - type: { [key: string]: Function | { get: Function, set: Function } }

## 源码

src/core/instance/state.js

依赖 [Watcher](../00-Vue/01-Vue-core/reactive/03-watcher.md) 的实现, 构造一个懒更新的 Watcher , computed 自身的 getter 触发时, 才去执行 watcher.evaluate() 得到最新值

- 首先给 Vue 实例添加一个 _computedWatchers 属性来存放计算属性对应的 Watcher 实例
- 取出用户定义的 getter , 构造一个懒更新的 Watcher
- 通过 createComputedGetter 函数构造一个新的 computedGetter , 负责在 _computedWatchers 取出相应的 watcher 并取值
- 用 Object.defineProperty 来定义计算属性, 使用上一步的 computedGetter 为 getter
- 如果在服务端渲染或指明不需要缓存, 则不需要用上述的 computedGetter, 只需用 createGetterInvoker 函数构造一个简单执行用户定义 getter 的函数作为 computedGetter

与 [props](./02-props.md) 一样, 定义计算属性时, 如果是已经存在的 key, 意味着这是在 [Vue.extend](../02-global-API/01-Vue.extend.md) 定义在原型上, 不需要再次定义 

```js
const computedWatcherOptions = { lazy: true }

function initComputed (vm: Component, computed: Object) {
  // $flow-disable-line
  const watchers = vm._computedWatchers = Object.create(null)
  // computed properties are just getters during SSR
  const isSSR = isServerRendering()

  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }

    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
}

export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : createGetterInvoker(userDef)
    sharedPropertyDefinition.set = noop
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : createGetterInvoker(userDef.get)
      : noop
    sharedPropertyDefinition.set = userDef.set || noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}

function createComputedGetter (key) {
  return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate()
      }
      if (Dep.target) {
        watcher.depend()
      }
      return watcher.value
    }
  }
}

function createGetterInvoker(fn) {
  return function computedGetter () {
    return fn.call(this, this)
  }
}
```