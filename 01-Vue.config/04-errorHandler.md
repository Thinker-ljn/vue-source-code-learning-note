# errorHandler

> 指定组件的渲染和观察期间未捕获错误的处理函数。
> - type: Function
> - default: undefined

组件运行过程中，在遇到错误时，调用 handlerError 函数，该函数会向上冒泡，在每一个祖先实例中调用 errorCaptured 钩子，如果钩子返回 false 则停止冒泡。否则最后会调用 globalHandleError 函数，这里就会尝试调用 config.errorHandler，如果用户没定义，则直接打印或拋出错误
