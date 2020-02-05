# silent

> 取消 Vue 所有的日志与警告。
> - type: boolean
> - default: false

## 初始定义

src/core/config.js

## 引用

src/core/util/debug.js

函数 warn, tip 定义中，当 Vue.config.silent 为真时，不打印错误

```js
warn = (msg, vm) => {
  const trace = vm ? generateComponentTrace(vm) : ''

  if (config.warnHandler) {
    config.warnHandler.call(null, msg, vm, trace)
  } else if (hasConsole && (!config.silent)) {
    console.error(`[Vue warn]: ${msg}${trace}`)
  }
}

tip = (msg, vm) => {
  if (hasConsole && (!config.silent)) {
    console.warn(`[Vue tip]: ${msg}` + (
      vm ? generateComponentTrace(vm) : ''
    ))
  }
}
```

## 相关

- debug