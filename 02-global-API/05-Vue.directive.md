# Vue.directive

> - 参数：
>   - {string} id
>   - {Function | Object} [definition]

> 注册或获取全局指令。

## 源码

/src/core/global-api/assets.js

没有传入第二个参数(definition)时, 返回相应 options.directives\[id] 的定义

如果 definition 是一个函数, 则包装成指令对象的形式, 然后放入相应 options.directives\[id]

Vue.component, Vue.filter 也是在这里定义

如果 component 方法传入的第二个参数是一个纯对象, 需要通过 Vue.extend 方法生成一个子类, 再放入相应的 options.components\[id]

```js
/* @flow */

import { ASSET_TYPES } from 'shared/constants'
import { isPlainObject, validateComponentName } from '../util/index'

export function initAssetRegisters (Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id)
        }
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id
          definition = this.options._base.extend(definition)
        }
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}

```