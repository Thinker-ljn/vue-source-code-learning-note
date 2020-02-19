# optimizer 优化器

优化器的目标：遍历生成的模板 AST 树并检测纯静态的子树，即永远不需要更改的DOM。一旦检测到这些子树，我们就可以：
1. 将它们提升为常量，这样我们就不再需要在每次重新渲染时为它们创建新的节点
2. 在 VNode Patch 过程中完全跳过它们
  
第一次遍历, 标记出**非静态**的节点
第二次遍历, 标记出**静态**的**根**节点

## 判断一个节点是否静态

节点有3种类型, 3 是文本节点, 一定是静态节点; 2 是表达式节点, 一定是非静态的; 1 是元素节点, 需要进一步判断

```js
function isStatic (node: ASTNode): boolean {
  if (node.type === 2) { // expression
    return false
  }
  if (node.type === 3) { // text
    return true
  }
  return !!(node.pre || (
    !node.hasBindings && // no dynamic bindings
    !node.if && !node.for && // not v-if or v-for or v-else
    !isBuiltInTag(node.tag) && // not a built-in
    isPlatformReservedTag(node.tag) && // not a component
    !isDirectChildOfTemplateFor(node) && // 父节点不是 (template 且有 v-for 指令的节点)
    Object.keys(node).every(isStaticKey)
  ))
}
```

## 标记出**非静态**的节点

- 递归地调用 `markStatic` 来标记非静态节点
  - 对本节点进行标记
  - 如果是元素节点
    - 非平台环境保留标签，且标签不为 `slot` ，且没有 `inline-template` 属性，返回，不标记
    - 递归对子节点标记，如果子节点是非静态的，则本节点也是非静态，重写这个属性
    - 对于条件语句生成的 `ifConditions` 属性，需要对 `ifConditions` 里面的所有 `block` 进行静态检查, 只要有任意一个 `block` 不是静态的, 那么当前的节点就是非静态的

## 标记出**静态**的**根**节点

- 递归调用 `markStaticRoots` 函数来标记
  - 目标节点应为元素节点
  - 对于静态节点或单次渲染节点(`v-once`), 执行 `node.staticInFor = isInFor`
  - 节点是静态的，且有子节点，且子节点不单单只是一个文本节点（这种情况没必要提升到静态渲染函数）
    - 标记为静态根
    - 否则不为静态根
  - 对子节点递归标记，传入 `node.for`
  - 递归对条件语句节点的 `block` 标记
