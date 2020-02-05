# Vue.version

> 提供字符串形式的 Vue 安装版本号。这对社区的插件和组件来说非常有用，你可以根据不同的版本号采取不同的策略。

## 源码

定义 src/core/index.js

```js
Vue.version = '__VERSION__'
```

在 scripts/config.js 获取 version

```js
const replace = require('rollup-plugin-replace')
const version = process.env.VERSION || require('../package.json').version

// ... 生成 rollup 配置
// built-in vars
const vars = {
  __WEEX__: !!opts.weex,
  __WEEX_VERSION__: weexVersion,
  __VERSION__: version
}
// ...

// 使用 rollup-plugin-replace 插件来替换 src/core/index.js 的 __VERSION__ 字符串
config.plugins.push(replace(vars))
// ...
```
