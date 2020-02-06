# lifecycle hooks 生命周期钩子

每个 Vue 实例在被创建时都要经过一系列的初始化过程. 例如, 需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等. 同时在这个过程中也会运行一些叫做生命周期钩子的函数, 这给了用户在不同阶段添加自己的代码的机会.

## hooks

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- activated
- deactivated
- beforeDestroy
- destroyed
- errorCaptured

## 源码

### callHook 方法 - 触发钩子的执行

src/core/instance/lifecycle.js 

- 使用 [pushTarget 和 popTarget](../00-Vue/01-Vue-core/reactive/02-dep.md) 来避免订阅依赖收集
- 在选项中获取相应的钩子回调, 依次执行

```js
export function callHook (vm: Component, hook: string) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget()
  const handlers = vm.$options[hook]
  const info = `${hook} hook`
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      invokeWithErrorHandling(handlers[i], vm, null, vm, info)
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}
```


### 各个钩子的调用时机

#### created / beforeCreate
src/core/instance/init.js

- Vue 实例初始化完生命周期, 事件, 渲染相关函数后, 触发 **beforeCreate** 钩子
- 接着初始化完实例数据相关的函数后, 触发 **created** 钩子
- 接着准备挂载 el

```js
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // ...
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    // ...
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
``` 
#### beforeMount / mounted / beforeUpdate
src/core/instance/lifecycle.js

- 当 Vue 实例挂载元素之前, 先触发 **beforeMount** 钩子
- 第一次渲染之后, 如果有 $vnode 的话, 触发 **mounted** 钩子
- 每当渲染订阅器的回调触发之前, 触发 **beforeUpdate** 钩子
- 接着准备执行 render 函数并更新 DOM

```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  // ...
  callHook(vm, 'beforeMount')
  // ...
  updateComponent = () => {
    vm._update(vm._render(), hydrating)
  }
  // ...
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false
  
  // ...
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

#### updated
src/core/observer/scheduler.js

在 [Watcher](../00-Vue/01-Vue-core/reactive/03-watcher.md) 源码分析知道渲染订阅器的回调默认是异步的, 会在 nextTick 的下一个更新周期执行

当异步队列里有渲染订阅器时, 就会触发 **updated** 钩子
```js
function flushSchedulerQueue () {
  // ...
  callUpdatedHooks(updatedQueue)
}
function callUpdatedHooks (queue) {
  let i = queue.length
  while (i--) {
    const watcher = queue[i]
    const vm = watcher.vm
    if (vm._watcher === watcher && vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'updated')
    }
  }
}
```

#### beforeDestroy / destroyed

src/core/instance/lifecycle.js

- 实例首次调用 $destroy 方法时, 先触发 **beforeDestroy** 钩子
- 执行完清理工作后, 触发 **destroyed** 钩子
- 相关的清理逻辑请查看 [$destroy](../12-instance-methods-lifecycle/04-vm.$destroy.md)

```js
Vue.prototype.$destroy = function () {
  const vm: Component = this
  if (vm._isBeingDestroyed) {
    return
  }
  callHook(vm, 'beforeDestroy')
  vm._isBeingDestroyed = true
  // -- isBeingDestroyed --
  callHook(vm, 'destroyed')
}
```

#### activated / deactivated

src/core/instance/lifecycle.js

keep-alive 组件激活/停用时调用, 详细分析暂缺.

```js
export function activateChildComponent (vm: Component, direct?: boolean) {
  if (direct) {
    vm._directInactive = false
    if (isInInactiveTree(vm)) {
      return
    }
  } else if (vm._directInactive) {
    return
  }
  if (vm._inactive || vm._inactive === null) {
    vm._inactive = false
    for (let i = 0; i < vm.$children.length; i++) {
      activateChildComponent(vm.$children[i])
    }
    callHook(vm, 'activated')
  }
}

export function deactivateChildComponent (vm: Component, direct?: boolean) {
  if (direct) {
    vm._directInactive = true
    if (isInInactiveTree(vm)) {
      return
    }
  }
  if (!vm._inactive) {
    vm._inactive = true
    for (let i = 0; i < vm.$children.length; i++) {
      deactivateChildComponent(vm.$children[i])
    }
    callHook(vm, 'deactivated')
  }
}
```

#### errorCaptured

参考 [handleError](../01-Vue.config/04-errorHandler.md)

## 相关参考

- [生命周期钩子文档](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)
