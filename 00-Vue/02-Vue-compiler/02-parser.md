# parser 解析器

解析器主要实现的功能是把 html 模板成 AST 节点树

## 编译的大体流程

- 编译器首先解析 html 模板成标签信息, 元信息再生成 ast 树
  - 解析 html 模板需要维护一个标签栈
    - 用相应的正则去匹配模板的标签信息, 遇到开始标签就解析标签名, 属性等并入栈, 结束标签就出栈, 除此之后还有文本信息等
  - 解析标签信息成为 ast, 也要维护一个 AST 栈
    - 每从 html 解析出一个(开始)标签信息(标签名, 属性列表等)时, 意味着这个标签的全部信息已经拿到, 由这些信息创建一个 AST 节点, 并且解析指令属性, 推入 AST 栈
    - 每从 html 解析出一个结束标签时, 将对应的 AST 节点出栈
    - 其他文本节点也有相应的处理

## 模板的解析过程

src/compiler/parser/html-parser.js

定义相应的正则变量, 用来匹配各种节点信息

### 解析过程 parseHTML 函数里的 while 过程
- 当 HTML string 不为空：
  - lastTag 为空或者不是**script,style,textarea**元素
    - 查找 **<** 字符
    - 如果是第一个字符，
      - 尝试匹配 Comment ,  如是就执行 **options.comment**，移动指针，进行下轮循环。
      - 不是则尝试匹配 ConditionalComment ,  如是则移动指针，进行下轮循环。
      - 不是则尝试匹配 Doctype ,  如是则移动指针，进行下轮循环。
      - 不是则尝试匹配 EndTag ,  如是则移动指针，解析 EndTag **parseEndTag**
      - 不是则 **parseStartTag**，成功则尝试匹配 StartTag , 如是 **handleStartTag**
    - 如果**不是**第一个字符
      - 用 slice 裁剪出由 **<** 字符开始到结尾的部分 rest 
      - 如果 找不到 endTag, startTagOpen, 并且不是 comment , 不是 conditionalComment
        - 就循环找下一个 **<** 字符，找不到就 break
        - 找到，把 **<** 字符之前的当成 text , 移动指针到 **<** 字符
    - 如果**找不到** **<** 字符
      - 把整个html string 当成 text
    - 处理 text **options.chars**
  - lastTag 是**script,style,textarea**元素
    - 解析出文本，赋予 text **options.chars**
- 执行最后的 **parseEndTag**, 清空栈

### advance(n) 函数
- index 移动 n 个单位
- html 裁剪掉前 n 个单位

### parseStartTag 函数
- html 匹配 startTagOpen 正则，
  - 成功则可以解析出 tagName, 开始位置，然后 advance 掉匹配的长度
  - 找出 startTagClose 和 attribute
  - 如果找到 startTagClose，记录 unarySlash（斜杠）位置，结束位置
  - 返回解析的 startTag 信息集合

### handleStartTag 函数

- 如果 expectHTML 为真
  - lastTag 是 'p' 并且当前 tagName 不是短语标签，结束 lastTag 标签 **parseEndTag**
  - tagName 是canBeLeftOpenTag 并且 lastTag === tagName，结束 lastTag 标签 **parseEndTag**
  - 处理 attrs
  - 如果 tagName 是 UnaryTag 或者有unarySlash
    - 把当前 tagName 信息入栈， lastTab = tagName
  - 处理 当前标签 **options.start**

### parseEndTag 函数

- 在栈中找出最靠近的同名的 tag 的位置
  - 如果位置大于等于 0 , 即找得到位置
    - 则 处理当前标签的结束 **options.end**
    - 出栈，lastTag = 栈顶的tag
  - 找不到, 如果 tagName === 'br'  **options.start**
  - 找不到, 如果 tagName === 'p'  **options.start** **options.end**


## AST 节点的生成

src/compiler/parser/index.js

### 变量说明

- inPre: 保留元素, 不编译

### 注释节点, 传入 ParseHTML 的 options.comment

节点类型 ID 是 3, 设置 isComment 属性为真, 和节点文本内容一起生成一个注释节点 ASTText, 放入当前的父节点的 children 属性


### 文本信息, 传入 ParseHTML 的 options.chars

- 普通的文本节点, 生成节点类型 ID 为 3 的文本节点
- 动态的文本节点, 
  - 由 src/compiler/parser/text-parser.js 处理
    - 匹配双花括号包裹的内容 exp, 生成 `_s(${exp})` 的 token
    - 其他分割的文本用双引用包裹
    - 按顺序用 `+` 连接生成一个表达式
  - 生成节点类型 ID 为 2, 带有表达式的文本节点

_s 是一个 Vue 实例的渲染方法, 用来把参数转为字符串, [详见这里](../../04-options-dom/03-render.md)

```js
const input = 'This is {{fruit}}, that is {{book}}'
const output = "This is" + _s(fruit) + ", that is" + _s(book)
```

### 元素节点 options.start - tag, attrs, unary, start, end

- 拿到 currentParent(父 ASTEle) 的namespace
- 如果有, 并且是 svg, 并且在IE, 处理attrs
- 创建一个 element: ASTEle
- 赋予 namespace
- 如果是禁止的元素标签: style, script(有text/javascript)
  - 设置 element.forbidden = true
- element **预变换**
- 如果 inVPre 不为真
  -  **processPre**
  - 如果 element.pre 为真或者是平台保留元素\<pre>, inVPre = true
- 如果 inVPre
  - **processRawAttrs**
- 不是
  - **processFor**
  - **processIf**
  - **processOnce**
- 如果 AST root 还没有设置, root = element
- 如果 !unary (不是一元标签)
  - currentParent = element
  - element 入栈
- 否则直接 **closeElement**

### 元素节点 options.end - tag, start, end

- 拿到栈顶 AST元素并出栈
- 设置 currentParent 为新的栈顶元素
- **closeElement**
