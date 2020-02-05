# Vue.compile

> - 参数：
>   - {string} template
> 在 render 函数中编译模板字符串。只在独立构建时有效

## 源码

定义在 src/platforms/web/entry-runtime-with-coompiler.js

具体实现请查看 compiler 相关

```js
import { compileToFunctions } from './compiler/index'
Vue.compile = compileToFunctions
```
