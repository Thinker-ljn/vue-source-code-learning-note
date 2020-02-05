# devtools

> 配置是否允许 vue-devtools 检查代码。
> - type: boolean
> - default: true  (生产版为 false)

浏览器环境下，获取 devtools 并初始化
```js
// detect devtools
export const devtools = inBrowser && window.__VUE_DEVTOOLS_GLOBAL_HOOK__
```

src/platforms/web/runtime/index.js
```js
if (config.devtools) {
  if (devtools) {
    devtools.emit('init', Vue)
  }
  // ...
}
```

src/core/observer/scheduler.js

刷新调度队列后，调用 devtools 接口更新数据。
```js
/**
 * Flush both queues and run the watchers.
 */
function flushSchedulerQueue () {
  // ...
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }
}
```

devtoolsMeta
