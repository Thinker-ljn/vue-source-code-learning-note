# watch

> - type: { [key: string]: string | Function | Object | Array }
> - 一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象

Vue 实例将会在实例化时遍历 watch 对象的每一个属性。解析出对应的对调函数, 调用实例方法 $watch

```js
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}

function createWatcher (
  vm: Component,
  expOrFn: string | Function,
  handler: any,
  options?: Object
) {
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  return vm.$watch(expOrFn, handler, options)
}
```