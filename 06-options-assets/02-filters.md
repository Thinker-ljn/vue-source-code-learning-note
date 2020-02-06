# filters 选项

过滤器的使用方式有两种: 双花括号插值和 v-bind 表达式, 都是在模板字符串上使用

```html
<!-- 在双花括号中 -->
<span>{{ message | capitalize }}</span>

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```

模板经过编译之后, 有过滤器的表达式, 会经下面的函数包装成一个函数调用, 具体的编译过程暂缺 // todo ...

src/compiler/parser/filter-parser.js
```js
function wrapFilter (exp: string, filter: string): string {
  const i = filter.indexOf('(')
  if (i < 0) {
    // _f: resolveFilter
    return `_f("${filter}")(${exp})`
  } else {
    const name = filter.slice(0, i)
    const args = filter.slice(i + 1)
    return `_f("${name}")(${exp}${args !== ')' ? ',' + args : args}`
  }
}
```

_f 函数是 Vue 实例原型上的方法, 由 src/core/instance/render-helpers/index.js 的 [installRenderHelpers](../04-options-dom/03-render.md) 函数定义, 最终 _f 调用了 [resolveAsset](./00-main.md) 函数来解析

```js
// resolve-filter.js
import { identity, resolveAsset } from 'core/util/index'
export function resolveFilter (id: string): Function {
  return resolveAsset(this.$options, 'filters', id, true) || identity
}

// index.js
import { resolveFilter } from './resolve-filter'
export function installRenderHelpers (target: any) {
  target._f = resolveFilter
}
```

