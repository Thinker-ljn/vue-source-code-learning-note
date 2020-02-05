# productionTip

> 设置为 false 以阻止 vue 在启动时生成生产提示。
> - type: boolean
> - default: false


src/platforms/web/runtime/index.js

```js
if (inBrowser) {
  //...
  if (process.env.NODE_ENV !== 'production' &&
      process.env.NODE_ENV !== 'test' &&
      config.productionTip !== false &&
      typeof console !== 'undefined'
    ) {
      console[console.info ? 'info' : 'log'](
        `You are running Vue in development mode.\n` +
        `Make sure to turn on production mode when deploying for production.\n` +
        `See more tips at https://vuejs.org/guide/deployment.html`
      )
    }
}
```