# Asset 资源

主要是三个选项: 
- directives 组件内指令数组
- filters 组件内过滤器函数数组
- components 组件内引用组件数组

当定义了这些选项时, 在 Vue 实例化时会与 Vue 构造函数或其父类构造函数的选择进行合并, 也就是说通过 [Vue.directive](../02-global-API/05-Vue.directive.md), [Vue.filter](../02-global-API/06-Vue.filter.md), [Vue.component](../02-global-API/07-Vue.component.md) 这三个全局 API 定义的资源, Vue 实例也可以访问.

## 如何调用

上述的全局资源定义后, 如果实例运行过程中需要解析对应的资源, 需调用下面的 resolveAsset 函数

src/core/util/options.js

```js
export function resolveAsset (
  options: Object,
  type: string,
  id: string,
  warnMissing?: boolean
): any {
  /* istanbul ignore if */
  if (typeof id !== 'string') {
    return
  }
  const assets = options[type]
  // check local registration variations first
  if (hasOwn(assets, id)) return assets[id]
  const camelizedId = camelize(id)
  if (hasOwn(assets, camelizedId)) return assets[camelizedId]
  const PascalCaseId = capitalize(camelizedId)
  if (hasOwn(assets, PascalCaseId)) return assets[PascalCaseId]
  // fallback to prototype chain
  const res = assets[id] || assets[camelizedId] || assets[PascalCaseId]
  if (process.env.NODE_ENV !== 'production' && warnMissing && !res) {
    warn(
      'Failed to resolve ' + type.slice(0, -1) + ': ' + id,
      options
    )
  }
  return res
}
```
