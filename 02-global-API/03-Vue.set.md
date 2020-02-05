# Vue.set

> Vue.set(target, key, value)
> - 参数: 
>   - {Object | Array} target
>   - {string | number} key
>   - {any} value
> - 返回 value

> - 向响应式对象添加一个属性, 确保这个新属性同样是响应式的, 可以触发视图更新
> - 该函数用来弥补 Vue observer 相关 API 的缺陷

## 源码

- 基本逻辑如下
  - 目标是数组：调用 splice 方法
  - 目标是对象：
    - 已有属性，直接赋值
    - 目标对象不是响应式，直接赋值
    - 如果是新属性，则定义成响应式
  
```js
export function set (target: Array<any> | Object, key: any, val: any): any {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot set reactive property on undefined, null, or primitive value: ${(target)}`)
  }
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    return val
  }
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  const ob = target.__ob__
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    )
    return val
  }
  if (!ob) {
    target[key] = val
    return val
  }
  defineReactive(ob.value, key, val)
  ob.dep.notify()
  return val
}
```