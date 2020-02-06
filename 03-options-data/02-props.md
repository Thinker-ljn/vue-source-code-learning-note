# props 数据对象

> - type: Array<\string> | Object
> - props 可以是数组或对象，用于接收来自父组件的数据。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选项，如类型检测、自定义校验和设置默认值。

## 源码

src/core/instance/state.js

### props 初始化入口
拿到 propsOptions, propsData, 循环对 propsOptions 的每个 key 在 propsData 取值, 然后定义成响应式

有两个点需要注意: 
- 除根组件外, 由于 props 的值是从父(祖先)组件传下的来, 意味着这个值已经被定义成响应式的了, 这里只要把 props 的 key 定义成响应式, 且只定义 get, 忽略 set, 因为 props 是不可写的
- 如果是静态的 props key, 意味着这是在 [Vue.extend](../02-global-API/01-Vue.extend.md) 定义的, 存在于原型上, 不需要通过 proxy 代理来访问这个静态的属性.

```js
function initProps (vm: Component, propsOptions: Object) {
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}
  // cache prop keys so that future props updates can iterate using Array
  // instead of dynamic object key enumeration.
  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent
  // root instance props should be converted
  if (!isRoot) {
    toggleObserving(false)
  }
  for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm)
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      const hyphenatedKey = hyphenate(key)
      if (isReservedAttribute(hyphenatedKey) ||
          config.isReservedAttr(hyphenatedKey)) {
        warn(
          `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
          vm
        )
      }
      defineReactive(props, key, value, () => {
        if (!isRoot && !isUpdatingChildComponent) {
          warn(
            `Avoid mutating a prop directly since the value will be ` +
            `overwritten whenever the parent component re-renders. ` +
            `Instead, use a data or computed property based on the prop's ` +
            `value. Prop being mutated: "${key}"`,
            vm
          )
        }
      })
    } else {
      defineReactive(props, key, value)
    }
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
  toggleObserving(true)
}
```

### props 的校验与取值

src/core/util/props.js 

这里的逻辑也比较清晰, 有个注意的点是取得默认值的时候, 需要对这个值进行观察, 定义成响应式. 同样的不能递归定义
