# Vue 选项数据的响应式处理

src/core/observe/index.js

## shouldObserve

Vue 定义了 shouldObserve 来显式的表明当前是否应该进行响应式处理, 通过 toggleObserving 来切换 shouldObserve 的值
```js
export let shouldObserve: boolean = true

export function toggleObserving (value: boolean) {
  shouldObserve = value
}
```

## Observer 类

Vue 定义了 Observer 类来包装需要响应式处理的 value
- 关联一个 [Dep](./02-dep.md) 实例, 用于 value 子孙属性的变更, 以及本身添加新属性或删除属性
- 记录引用这个 value 的 Vue 实例的个数
- 把自身赋予 value 的 \__ob__ 属性
- 对 value 进行响应式处理

```js
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

### vmCount 属性

在调用 vm.$destroy 时会进行相应的减数操作

### 数组的响应式处理

- 以 Array.prototype 为原型, 创建一个 arrayMethods 变量
- 改造 arrayMethods , 对于可以改变数组本身的数组 API 如 push, pop, splice 等进行重写, 调用这些 API 时, 需要通过对应的订阅, 如果是需要传入参数的 API 如 splice, unshift 等, 需要对传入的参数进行响应式处理
- 如果平台环境可以使用 \__proto__ 属性, 那么 value.\__proto__ = arrayMethods
- 否则, 把 arrayMethods 的属性值复制到 value, 来重写数组方法
- 最后调用 observeArray 来循环递归处理 value 的子项, 这里的每一个子项都需要进行响应式处理

### 对象的响应式处理

循环把对象的每一个属性通过 defineReactive 函数都处理为响应式

#### defineReactive 函数

该函数是响应式的核心, 通过 Object.defineProperty 方法来重写属性的 get, set 特性, 以便在属性被获取或修改时, 进行相应的逻辑操作. 

- 定义一个 [Dep](./02-dep.md) 实例 dep, 用于该属性的订阅收集
- 对属性对应的值进行响应式处理, 令返回的 Observer 对象赋予 childOb, 也就是说, 值是对象的话就递归地对该属性的子孙属性进行处理
- get: 属性被调用时
  - 获取对应的属性值 value
  - 如果有 Dep.target, 这是当前调用该属性的订阅器 Watcher
    - 调用 dep.depend() 添加这个订阅器
    - 如果有 childOb, 调用 childOb.dep.depend(), 因为订阅一个对象, 对象里的任意子孙属性的变更都需要通知订阅者
    - 如果 value 是一个数组, 要递归获取数组里的所有 Observer 对象, 调用相应的 dep.depend()
- set: 属性被修改时
  - 如果修改的值与旧值一致, 则不处理
  - 对新的值进行响应式处理
  - 触发 dep.notify , 通知所有订阅该属性的订阅器更新

## observe 函数

- Vue 定义了 observe 函数来对传入的 value 参数添加响应式对象 Observer
  - 如果 value 不是对象或者是 VNode 的实例, 返回, 不处理
  - 如果 value 本身拥有属性 \__ob__ , 并且对应的值是一个 Observer 实例, 返回这个对象
  - 如果 shouldObserve 为真, 且不是服务端渲染, 且 value 是数组或对象, 且 value 是可扩展的(可添加新属性), 且 value 不是 Vue 实例
    - 以 value 为参数创建一个 Observer 实例 (执行后, 这个实例会赋予 value.\__ob__)
  - 如果当前的 value 是 Vue 实例的根数据
    - value.\__ob__.vmCount++
  - 返回 value.\__ob__


## 缺陷与弥补

Vue 对选项数据的响应式处理是通过 Object.defineProperty 来劫持属性的获取与修改, 这就意味着对于新增属性与删除属性是无法响应的.

所以实现了 [Vue.set](../../../02-global-API/03-Vue.set.md), [Vue.delete](../../../02-global-API/04-Vue.delete.md) 来显示的触发响应

### 数组响应式的变通

- 数组的 length 不能 defineProperty - non-configurable，non-enumerable

- 数组下的索引是可以用setter和getter的，但是为啥vue下索引也不允许更新呢？
  - 因为length = 5的数组，未必索引就有4，这个索引(属性)不存在，就没法setter了。
  - 另外, 如果对每个索引进行观察,那么当使用 unshift 等会改变所有索引的 api 时,将会通知所有的订阅者,哪怕订阅的数组项并没有改变

- [记一次思否问答的问题思考：Vue为什么不能检测数组变动](https://segmentfault.com/a/1190000015783546)

