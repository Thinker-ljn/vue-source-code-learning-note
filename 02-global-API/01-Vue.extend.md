# Vue.extend 函数

> Vue.extend( options )
> 接收一个 含 Vue 组件选项的对象参数，返回一个继承基础 Vue 构造器的子类
> 其中，data 选项必须是函数

## 源码

定义在 src/core/global-api/extend.js, 扩展 Vue 的全局 API 时被安装

### 基本的寄生组合继承

```js 
Vue.extend = function (extendOptions) {
  // ...
  const Super = this
  // ...
  const Sub = function VueComponent (options) {
    this._init(options) // 调用父类的初始化函数
  }
  Sub.prototype = Object.create(Super.prototype) // 继承原型
  Sub.prototype.constructor = Sub // 绑定构造函数
  // ...
}
```
### 唯一性与缓存

对于每一个子类, 创建一个 cid (自增)属性来保证唯一性, 并且可以用来作为缓存 id

子类通过继承也会有 extend 方法, 继承时由 mergeOptions 方法也继承了父类的 options, 包括了用来记录缓存的 _Ctor 属性

```js 
Vue.cid = 0
let cid = 1
Vue.extend = function (extendOptions) {
  // ...
  const Super = this // 父类, 由于子类也会有 extend 方法, 这里不一定是基础 Vue 构造函数 
  const SuperId = Super.cid
  const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
  if (cachedCtors[SuperId]) {
    return cachedCtors[SuperId]
  }
  // ...
  Sub.cid = cid++
  Sub.options = mergeOptions(
    Super.options,
    extendOptions
  )
  // ...
  cachedCtors[SuperId] = Sub
}
```

### 静态属性与方法的继承

对于 props 和 computed 选项, 为了避免在每次创建实例时都通过 Object.defineProperty 定义成私有的变量, 提前在类继承时就把它们定义在子类的原型上, 成为实例共享的属性

```js
// ...
if (Sub.options.props) {
  initProps(Sub)
}
if (Sub.options.computed) {
  initComputed(Sub)
}
// ...
function initProps (Comp) {
  const props = Comp.options.props
  for (const key in props) {
    proxy(Comp.prototype, `_props`, key)
  }
}

function initComputed (Comp) {
  const computed = Comp.options.computed
  for (const key in computed) {
    defineComputed(Comp.prototype, key, computed[key])
  }
}
```

其他的属性简单的复制引用关系

```js
Sub.extend = Super.extend
Sub.mixin = Super.mixin
Sub.use = Super.use

// create asset registers, so extended classes
// can have their private assets too.
ASSET_TYPES.forEach(function (type) {
  Sub[type] = Super[type]
})
// enable recursive self-lookup
if (name) {
  Sub.options.components[name] = Sub
}

// keep a reference to the super options at extension time.
// later at instantiation we can check if Super's options have
// been updated.
Sub.superOptions = Super.options
Sub.extendOptions = extendOptions
Sub.sealedOptions = extend({}, Sub.options)
```