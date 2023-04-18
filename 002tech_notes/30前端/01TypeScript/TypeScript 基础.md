## 1 相关文档
https://www.bilibili.com/video/BV1NR4y1x7Ab?p=1

https://24kcs.github.io/vue3_study/

## 2 初识TS
TypeScript是一种由微软开发的开源、跨平台的编程语言。它是JavaScript的超集，最终会被编译为JavaScript代码。
**TypeScript 是 JavaScript 的一个超集**，主要提供了**类型系统**和**对 ES6+ 的支持**，它由 Microsoft 开发，代码[开源于 GitHub](https://github.com/Microsoft/TypeScript)上。

### 2.1 TypeScript 3 大特点：
-   **始于JavaScript，归于JavaScript**
TypeScript 可以编译出纯净、 简洁的 JavaScript 代码，并且可以运行在任何浏览器上、Node.js 环境中和任何支持 ECMAScript 3（或更高版本）的JavaScript 引擎中。

-   **强大的类型系统**
**类型系统**允许 JavaScript 开发者在开发 JavaScript 应用程序时使用高效的开发工具和常用操作比如静态检查和代码重构。

-   **先进的 JavaScript**
TypeScript 提供最新的和不断发展的 JavaScript 特性，包括那些来自 2015 年的 ECMAScript 和未来的提案中的特性，比如异步功能和 Decorators，以帮助建立健壮的组件。

## 3 基本语法
### 3.1 类型注解
```typescript
// greeter.ts 文件
function greeter (person: string) {
  return 'Hello, ' + person
}

let user = 'Yee'

console.log(greeter(user))
```
上面的代码中 `person: string`使用的类型z

要注意的是尽管有错误，`greeter.js` 文件还是被创建了。 就算你的代码里有错误，你仍然可以使用 TypeScript。但在这种情况下，TypeScript 会警告你代码可能不会按预期执行。
