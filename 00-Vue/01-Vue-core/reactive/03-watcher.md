# Watcher 类 

订阅器接收 expression 并解析成 getter, 触发 getter 的时候收集依赖, 如果有回调参数的话, 就执行并传入 getter 返回的值

## getter 初始化

- 如果实例化 Watcher 的第二个参数 expOrFn 是函数
  - 直接赋予 getter
- 否则调用 parsePath 解析, 解析不成功则 getter 指向空函数

### parsePath 函数 
```js
/**
 * unicode letters used for parsing html tags, component names and property paths.
 * using https://www.w3.org/TR/html53/semantics-scripting.html#potentialcustomelementname
 * skipping \u10000-\uEFFFF due to it freezing up PhantomJS
 */
export const unicodeRegExp = /a-zA-Z\u00B7\u00C0-\u00D6\u00D8-\u00F6\u00F8-\u037D\u037F-\u1FFF\u200C-\u200D\u203F-\u2040\u2070-\u218F\u2C00-\u2FEF\u3001-\uD7FF\uF900-\uFDCF\uFDF0-\uFFFD/

const bailRE = new RegExp(`[^${unicodeRegExp.source}.$_\\d]`)
export function parsePath (path: string): any {
  if (bailRE.test(path)) {
    return
  }
  const segments = path.split('.')
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]]
    }
    return obj
  }
}
```

## getter 的执行 - watcher.get()

- 调用 pushTarget 把实例本身放入 Dep 的订阅栈
- 调用 getter 取值, 触发所依赖数据的 get 特性, 收集依赖
- 处理深度订阅
- 调用 popTarget 出栈
- 更新依赖数组
- 返回 getter 的返回值

### deep option

如果 watcher.deep 为真, 需要对 getter 的返回值进行深层遍历

如果遍历到的属性值是拥有 \__ob__ 属性, 且 ob.dep.id 是已经遍历过的, 就跳过, 不需要遍历第二次

为何要这么做? 所谓遍历就是查看属性值, 也就是取值, 就会触发响应式属性的 getter , 这样就能把深度订阅的值的所有子孙属性都添加当前的订阅器

## 什么时候触发 getter - update / run / evaluate 方法

- 当订阅器的某个依赖更新时, 会触发 update 方法, 三种执行模式
- 懒更新: watcher.lazy 为真, 则先置 watcher.dirty 为真, 等待 evaluate 方法的执行. 
- 同步更新: watcher.sync 为真, 直接执行 run 方法
- 默认异步更新: 把当前 watcher 推入队列, 在下一个 nextTick 更新周期清空队列时执行每一个 watcher 的 run 方法, [具体分析](./04-scheduler.md)

## watcher 的销毁

- 把实例自身移出 Vue 实例的 _watchers 数组
- 把实例自身移出依赖的订阅数组
- 把 watcher.active 置为假 


## 使用场景

render watcher

computed options

watch options

$watch method