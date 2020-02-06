# renderError 选项

> - type: (createElement: () => VNode, error: Error) => VNode
> - 只在开发者环境下工作。当 render 函数遭遇错误时，提供另外一种渲染输出。其错误将会作为第二个参数传递到 renderError。这个功能配合 hot-reload 非常实用。

源码分析请查看 [render](./03-render.md)
