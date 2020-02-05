# data 数据对象

>- type: Object | Function (组件的定义只接受 Function)
>- Vue 实例的数据对象。Vue 将会递归将 data 的属性转换为 getter/setter，从而让 data 的属性能够响应数据变化。

## 源码

src/core/instance/state.js

初始化 data 选项时, 如果 data 是一个函数, 则调用函数获取对象, 再检查属性冲突, 然后调用 [observe](../00-Vue/01-Vue-core/reactive/01-observe.md) 方法观察 data 值

```js
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  
  // -- 检查是否返回一个对象

  // -- 检查是否与 methods props 的属性冲突

  observe(data, true /* asRootData */)
}
```

在调用 data 函数时, 使用了 getData 的方法, 利用 pushTarget 将 Dep 维护的栈顶置为 undefined, 以避免在获取 data 对象时触发依赖收集. 

pushTarget, Dep 相关的说明请查看 [Dep](../00-Vue/01-Vue-core/reactive/02-dep.md)

```js
export function getData (data: Function, vm: Component): any {
  // #7573 disable dep collection when invoking data getters
  pushTarget()
  try {
    return data.call(vm, vm)
  } catch (e) {
    handleError(e, vm, `data()`)
    return {}
  } finally {
    popTarget()
  }
}
```