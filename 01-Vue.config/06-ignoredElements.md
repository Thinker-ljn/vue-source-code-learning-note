# ignoredElements

> 使 Vue 忽略在 Vue 之外的自定义元素。
> - type: Array<string | RegExp>
> - default: []

当 Vue 渲染组件时，遇到没有注册的组件名字，会抛出一个 Unknown custom element 的警告，一般来说是用户忘记注册。

如果用户必须如此编写（如 Web Components APIs），就可以使用这个选项来避免警告。

源码中，主要是在 vdom 相关的代码中 /src/core/vdom/patch.js 使用 isUnknownElement 检查。