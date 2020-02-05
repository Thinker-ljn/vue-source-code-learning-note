# keyCodes

> 给 v-on 自定义键位别名。
> - type: {[key: string]: number | Array<number>}
> - default: {}

Vue 内置了 keyCodes 来检测用户编写的 v-on 的键值, 用户可通过自定义来扩展

## 源码

src/core/instance/render-helpers/check-keycodes.js

用来检测 keyCode 是否是正确的
```js
// expect 不包含 actual 或 不等于 actual
function isKeyNotMatch (expect, actual): boolean

export function checkKeyCodes (
  eventKeyCode: number,  // 原生 Event.keyCode
  key: string,           // v-on 指令传入的 key
  builtInKeyCode?: number | Array<number>, // Vue 内置对应的 keyCode
  eventKeyName?: string,                   // 原生 Event.key 
  builtInKeyName?: string | Array<string>  //  Vue 内置对应的 keyName
): ?boolean {
  const mappedKeyCode = config.keyCodes[key] || builtInKeyCode
  if (builtInKeyName && eventKeyName && !config.keyCodes[key]) {
    return isKeyNotMatch(builtInKeyName, eventKeyName)
  } else if (mappedKeyCode) {
    return isKeyNotMatch(mappedKeyCode, eventKeyCode)
  } else if (eventKeyName) {
    return hyphenate(eventKeyName) !== key
  }
}
```
src/compiler/codegen/events.js 定义了内置的 keyCodes

```js
// KeyboardEvent.keyCode aliases
const keyCodes: { [key: string]: number | Array<number> } = {
  esc: 27,
  tab: 9,
  enter: 13,
  space: 32,
  up: 38,
  left: 37,
  right: 39,
  down: 40,
  'delete': [8, 46]
}
const keyNames: { [key: string]: string | Array<string> } = {
  // #7880: IE11 and Edge use `Esc` for Escape key name.
  esc: ['Esc', 'Escape'],
  tab: 'Tab',
  enter: 'Enter',
  // #9112: IE11 uses `Spacebar` for Space key name.
  space: [' ', 'Spacebar'],
  // #7806: IE11 uses key names without `Arrow` prefix for arrow keys.
  up: ['Up', 'ArrowUp'],
  left: ['Left', 'ArrowLeft'],
  right: ['Right', 'ArrowRight'],
  down: ['Down', 'ArrowDown'],
  // #9112: IE11 uses `Del` for Delete key name.
  'delete': ['Backspace', 'Delete', 'Del']
}
```