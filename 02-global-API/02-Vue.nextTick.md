# Vue.nextTick

> - Vue.nextTick( [callback, context] )
> - 在下次 DOM 更新循环结束之后执行延迟回调, 没有提供回调时, 返回一个 Promise (环境支持的话)
> - Vue 实例中 $nextTick 也是指向了这个函数

## 源码

定义在 /src/core/util/next-tick.js

首先定义一个回调队列, 队列处理状态 pending, 一个清空队列的函数

清空队列时, pending = false
```js
export let isUsingMicroTask = false

const callbacks = []
let pending = false

function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
```

在不同的平台环境下, 用不同的方法实现 timerFunc, 并解决不同平台下的特殊情况.

依次检测 Promise MutationObserver setImmediate setTimeout, 也就是优化使用能产生微任务的 API, 实在不行就使用 setTimeout

timerFunc 的回调里执行 flushCallbacks 来清空队列
```js
let timerFunc
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

调用 nextTick 时, 生成一个回调函数, 放入回调队列.

定义一个 _resolve 方法, 在没有传入自定义回调时, 生成一个 Promise 并且把 resolve 赋予 _resolve

回调函数在队列清空时执行, 要么执行自定义回调, 要么执行 _resolve, 或者跳过

当 pending 不为真, 表示上一周期的队列已经清空, 此时把 pending 置为 true, 调用 timerFunc 来触发下一周期的清空队列的回调 

```js
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```