# components 选项

当 Vue 实例调用 render 函数时, 会传入一个 createElenment 函数, 用来渲染 VNode, 其中会尝试调用 [resolveAsset](./00-main.md) 来获取组件定义并创建

src/core/vdom/create-element.js

```js
// ... 
if (typeof tag === 'string') {
  let Ctor
  ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
  if (config.isReservedTag(tag)) {
    // platform built-in elements
    if (process.env.NODE_ENV !== 'production' && isDef(data) && isDef(data.nativeOn)) {
      warn(
        `The .native modifier for v-on is only valid on components but it was used on <${tag}>.`,
        context
      )
    }
    vnode = new VNode(
      config.parsePlatformTagName(tag), data, children,
      undefined, undefined, context
    )
  } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
    // component
    vnode = createComponent(Ctor, data, context, children, tag)
  } else {
    // unknown or unlisted namespaced elements
    // check at runtime because it may get assigned a namespace when its
    // parent normalizes children
    vnode = new VNode(
      tag, data, children,
      undefined, undefined, context
    )
  }
} else {
  // direct component options / constructor
  vnode = createComponent(tag, data, context, children)
}
```