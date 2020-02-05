# warnHandler

> 开发者环境下为 Vue 的运行时警告赋予一个自定义处理函数。
> - type: Function
> - default: undefined

当调用了 src/core/util/debug.js 的 warn 函数时，如果有用户自定义的 warnHandler ，则优先调用。

## 相关

- [silent](./01-silent.md)
