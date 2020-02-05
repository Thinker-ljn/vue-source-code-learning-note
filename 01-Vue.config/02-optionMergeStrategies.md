# optionMergeStrategies

> 自定义合并策略的选项。
> - type: { [key: string]: Function }
> - default: {}

## 初始定义

src/core/config.js

## 合并策略

Vue 实例化/继承/混合的过程中，需要合并两个对象选项，基本的合并策略是由后置（子对象）的值覆盖前面（父对象）的值。如果没有后置的值，直接返回前置的值。

在 src/core/util/options.js 内，还定义了其他内置的合并策略

如果设置了 optionMergeStrategies，则对应的内置策略会被自定义的函数覆盖。

### 实例中才能合并的选项

对于 el，propsData ，在 Vue 实例化后才有效，在开发模式下会警告

```js
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(...)
    }
    return defaultStrat(parent, child)
  }
}
```

### data 选项

定义了一个 mergedDataFn 返回一个合并策略函数，如果 data 是一个函数，则用 vm(当前实例，没有则用 this 值) 来调用 data 函数，然后用 mergeData 函数还递归调用合并两个数据。

在 mergeData 中，跳过响应式 \__ob__ 的 key，对于被合并对象没有的 key ，用 set 函数来赋值。

参考响应式相关的说明。

### hook 相关

在 src/shared/constants.js 定义了 Vue 生命周期的名称

```js
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured',
  'serverPrefetch'
] 
```

生命周期的 hook 的值是函数数组，在合并时，需要保证返回的是一个数组，且数组内的函数的唯一。

### component directive filter 方法

通过 extend 函数合并

src/shared/util.js extend function 相当于 Object.assign

### watch

通过 extend 函数合并，不过 watch 的值不能在合并时修改它们，需要处理成数组，因为每一个watch 都可以拥有多个回调。

### props methods inject computed 

通过 extend 函数合并

### provide 

同 data，使用了 megeDataOrFn

