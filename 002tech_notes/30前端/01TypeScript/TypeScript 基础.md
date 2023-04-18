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
**类型系统**允许 JavaScript 开发者在开发 JavaScript 应用程序时使用高效的开发工具和常用操作比如==静态检查==和代码重构。

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

console.log(greeter(user))        // 检测正常
console.log(greeter([1,2,3]))     // 这里会报错

```
上面的代码中 `person: string`使用的类型注解。表示了这个函数只能接受 string 类型的入参。
以其他类型的参数作为入餐，会导致使用`tsc`编译时报错。要注意的是尽管有错误，`greeter.js` 文件还是被创建了。 就算你的代码里有错误，你仍然可以使用 TypeScript。但在这种情况下，TypeScript 会警告你代码可能不会按预期执行。


## 4 范式
### 4.1 (() => ( )) ()这样的结构是代表什么意思
在 TypeScript 中，`() => ()` 表示一个没有参数、没有返回值的函数类型。`() => (expr)` 则表示一个没有参数，返回类型为表达式 `expr` 类型的函数类型。

而 `(() => ())()` 这样的范式表示先定义了一个没有参数、没有返回值的函数，然后对这个函数进行立即调用。通常这种写法被称为 IIFE（Immediately Invoked Function Expression）或自执行函数。

例如，下面的代码中，IIFE 在函数内部定义了变量 `x` 并返回了一个新的值：
```typescript
const result = (() => {
  let x = 10;
  return x * 2;
})();
console.log(result); // 输出 20
```

**IIFE 常用于创建一个独立的作用域，避免变量污染全局作用域，并且可以在程序初始化时执行某些代码逻辑。**

