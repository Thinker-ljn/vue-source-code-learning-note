# global-API

定义在 src/core/global-api/index.js

在这里, 定义了全局的属性和方法, 包括了之前提到的 Vue.config

简化后的代码如下

```js
// vm.constructor === Vue
// vm.constructor.options = Vue.options

// Vue.config

// Vue.util 不推荐作用公有的 API 
Vue.util = {
  warn,
  extend,
  mergeOptions,
  defineReactive
}

Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick

const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
ASSET_TYPES.forEach(type => {
  Vue.options[type + 's'] = Object.create(null)
})
// Vue.options.components = {}
// Vue.options.directives = {}
// Vue.options.filters = {}
// Vue.options._base = Vue

extend(Vue.options.components, builtInComponents) // KeepAlive

initUse(Vue)
initMixin(Vue)
initExtend(Vue)
initAssetRegisters(Vue)
// Vue.use = function () {}
// Vue.mixin = function () {}
// Vue.extend = function () {}
// Vue.component = function (id, definition) {}
// Vue.directive = function (id, definition) {}
// Vue.filter = function (id, definition) {}
``` 