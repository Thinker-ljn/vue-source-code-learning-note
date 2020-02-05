# Vue.config

## 初始定义

src/core/config.js

## 引用

src/core/global-api/index.js

Vue 实例初始化之前先对 Vue 类初始化全局 API，Vue.config 是一个只读属性，用 Object.defineProperty 来定义其 get，忽略 set

```js
import config from '../config'
// ...
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)
}
```
