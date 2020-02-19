# compiler options

- 创建编译器时传入的 baseOptions
- 执行编译时传入的 options

## CompilerOptions 类型

flow/compiler.js

```js
declare type CompilerOptions = {
  warn?: Function; // 允许在不同的平台环境自定义警告函数 e.g. node
  modules?: Array<ModuleOptions>; // 平台环境的特殊模块; e.g. style; class
  directives?: { [key: string]: Function }; // 平台环境的特殊指令
  staticKeys?: string; // 被看作是静态的AST属性列表; 用于优化
  isUnaryTag?: (tag: string) => ?boolean; // 检查标签对于当前平台是不是一元标签
  canBeLeftOpenTag?: (tag: string) => ?boolean; // check if a tag can be left opened
  isReservedTag?: (tag: string) => ?boolean; // 检查标签对于当前平台是不是原生的
  preserveWhitespace?: boolean; // preserve whitespace between elements? (Deprecated)
  whitespace?: 'preserve' | 'condense'; // whitespace handling strategy
  optimize?: boolean; // optimize static content?

  // web specific
  mustUseProp?: (tag: string, type: ?string, name: string) => boolean; // check if an attribute should be bound as a property
  isPreTag?: (attr: string) => ?boolean; // check if a tag needs to preserve whitespace
  getTagNamespace?: (tag: string) => ?string; // check the namespace for a tag
  expectHTML?: boolean; // only false for non-web builds
  isFromDOM?: boolean;
  shouldDecodeTags?: boolean;
  shouldDecodeNewlines?:  boolean;
  shouldDecodeNewlinesForHref?: boolean;
  outputSourceRange?: boolean;

  // runtime user-configurable
  delimiters?: [string, string]; // template delimiters
  comments?: boolean; // preserve comments in template

  // for ssr optimization compiler
  scopeId?: string;
};
```
## baseOptions
### 模块与指令

不同平台的模块与指令的定义, 用于 codegen 阶段把用户编写的模板模块与指令转为 js 代码

以 web 平台为例，会有编译时和运行时的模块或指令，在这里自然只用到 compiler 文件夹下的模块或指令。运行时的会在 VNode Patch 时使用。

- src/platforms/web
  - compiler
    - directives
      - html.js
      - model.js
      - text.js
    - modules
      - class.js
      - model.js
      - style.js
    - options.js
  - runtime
    - components
      - transition-group.js
      - transition.js
    - directives
      - model.js
      - show.js
    - modules
      - attrs.js
      - class.js
      - dom-props.js
      - events.js
      - style.js
      - transition.js
