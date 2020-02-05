# Dep 类

为每一个被观察的对象, 属性收集相应的订阅

src/core/observe/dep.js

## Dep.target

静态属性 Dep.target 用来表示当时正在执行的订阅器 Watcher

Dep 文件定义了一个订阅栈, 栈顶的值就是 Dep.target, 订阅器执行时, 可能会调用其他订阅器, 这是一个栈结构, 就如同 js 调用函数一样, 需要一个函数调用栈.

当 Dep.target 是假值时, 表示当前没有订阅器或者不需要进行订阅收集

```js
Dep.target = null
const targetStack = []

export function pushTarget (target: ?Watcher) {
  targetStack.push(target)
  Dep.target = target
}

export function popTarget () {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}
```

## depend 方法与 addSub 方法

在 observe 里, 收集订阅就是调用这个方法, Dep.target 是 Watcher 实例时, 调用 watcher.addDep, 把自身(dep 实例)添加到订阅器的依赖列表中. 

查看 Watcher 的源码可以知道, watcher.addDep 的执行又会反过来调用 Dep 的 addSub 方法, 把订阅器自身实例添加到 dep 的订阅列表

```js
class Dep {
  // ...
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  // ... 
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  // ...
}
```

## notify 方法

按顺序执行所有订阅的 update 方法
