# Vue.mixin

> - 参数：
>   - {Object} mixin
>  全局注册一个混入，影响注册之后所有创建的每个 Vue 实例。插件作者可以使用混入，向组件注入自定义的行为。不推荐在应用代码中使用。

## 源码

mixin 的实现只是简单的包装了 mergeOptions, 具体的实现可查看 [optionMergeStrategies](../01-Vue.config/02-optionMergeStrategies.md)

```js
import { mergeOptions } from '../util/index'

export function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    this.options = mergeOptions(this.options, mixin)
    return this
  }
}
```
