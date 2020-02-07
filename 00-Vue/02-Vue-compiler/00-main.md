# compiler

src/compiler

- 在这个文件夹下, 最终是导出一个 createCompiler, 传入不同平台环境的 baseOptions 创建编译器的函数。代码嵌套较多, 有点绕, 主体的逻辑如下
  - createCompiler 返回 { compile, compileToFunctions }
    - compile 是把模板字符串编译成 { ast, render, staticRenderFns }, render, staticRenderFns 是函数代码字符串
    - compileToFunctions 是把上述的 render, staticRenderFns 转化为可调用的函数
  - 不同平台的代码主要是用 compileToFunctions 来生成 render, staticRenderFns


## 入口
首先在入口 index.js 下, createCompiler 是由 createCompilerCreator 生成, createCompilerCreator 需要一个参数 baseCompile 函数。该函数逻辑比较清晰, 解析模板字符串成 AST, 优化解析的 AST 结果, 由 AST 生成代码并返回

稍微变形下代码：
```javascript
import { parse } from './parser/index'
import { optimize } from './optimizer'
import { generate } from './codegen/index'
import { createCompilerCreator } from './create-compiler'

function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  if (options.optimize !== false) {
    optimize(ast, options)
  }
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
}

export const createCompiler = createCompilerCreator(baseCompile)
```

## 创建一个编译器生成函数

src/compiler/create-compiler.js

createCompilerCreator (顾名思义:创建一个能创建编译器的函数), 函数返回一个能接收 baseOptions (不同平台环境的选项)的 createCompiler(创建不同平台环境的编译器的函数), 生成一个包装了 baseCompile 的 compile 函数

compile 函数会合并 baseOptions 与 options(由编译时传入给 compile), 调用 baseCompile 函数执行编译逻辑返回编译结果, 结果中的 render 函数是一个字符串代码, 而不是可调用的函数

所以 createCompiler 函数还调用 createCompileToFunctionFn 函数把 compile 函数包装成一个能生成可调用的 render 函数的编译函数 (包括了静态 render 函数)


## 创建一个能生成 render 函数的编译函数 - createCompileToFunctionFn

src/compiler/to-functions.js
 
主体逻辑是 compile 生成 render 函数字符串代码后, 使用 new Function(renderCode) 来转为函数

这里面还用到了缓存, 以 options.delimiters + template 为键来存储编译结果
