# Vue.delete

> Vue.delete(target, key)
> - 参数: 
>   - {Object | Array} target
>   - {string | number} key

> 删除对象的属性。如果对象是响应式的，确保删除能触发更新视图
> 该函数用来弥补 Vue observer 相关 API 的缺陷

## 源码

- 基本逻辑如下
  - 目标是数组：调用 splice 方法
  - 目标是对象：
    - 不是目标对象自身的属性(比如是原型的), 直接返回
    - 用 delete 关键字删除属性
    - 目标对象是响应式, 则通知相关的依赖

```js
export function del (target: Array<any> | Object, key: any) {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot delete reactive property on undefined, null, or primitive value: ${(target)}`)
  }
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1)
    return
  }
  const ob = target.__ob__
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid deleting properties on a Vue instance or its root $data ' +
      '- just set it to null.'
    )
    return
  }
  if (!hasOwn(target, key)) {
    return
  }
  delete target[key]
  if (!ob) {
    return
  }
  ob.dep.notify()
}

```