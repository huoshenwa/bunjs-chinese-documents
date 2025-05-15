JavaScript 中的模块解析是一个复杂的主题。

生态系统目前正处于从 CommonJS 模块到原生 ES 模块的多年过渡期。TypeScript 强制执行其自己的一套关于导入扩展的规则，这些规则与 ESM 不兼容。不同的构建工具通过不同的不兼容机制支持路径重映射。

Bun 旨在提供一个一致且可预测的模块解析系统，使其"即插即用"。不幸的是，它仍然相当复杂。

## 语法

考虑以下文件。

{% codetabs %}

```ts#index.ts
import { hello } from "./hello";

hello();
```

```ts#hello.ts
export function hello() {
  console.log("Hello world!");
}
```

{% /codetabs %}

当我们运行 `index.ts` 时，它打印 "Hello world!"。

```bash
$ bun index.ts
Hello world!
```

在这种情况下，我们从 `./hello` 导入，这是一个没有扩展名的相对路径。**带扩展名的导入是可选的，但受支持。** 要解析这个导入，Bun 将按顺序检查以下文件：

- `./hello.tsx`
- `./hello.jsx`
- `./hello.ts`
- `./hello.mjs`
- `./hello.js`
- `./hello.cjs`
- `./hello.json`
- `./hello/index.tsx`
- `./hello/index.jsx`
- `./hello/index.ts`
- `./hello/index.mjs`
- `./hello/index.js`
- `./hello/index.cjs`
- `./hello/index.json`

导入路径可以选择包含扩展名。如果存在扩展名，Bun 将只检查具有该确切扩展名的文件。

```ts#index.ts
import { hello } from "./hello";
import { hello } from "./hello.ts"; // 这可以工作
```

如果您从 `"*.js{x}"` 导入，Bun 将额外检查匹配的 `*.ts{x}` 文件，以兼容 TypeScript 的 [ES 模块支持](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html#new-file-extensions)。

```ts#index.ts
import { hello } from "./hello";
import { hello } from "./hello.ts"; // 这可以工作
import { hello } from "./hello.js"; // 这也可以工作
```

Bun 支持 ES 模块（`import`/`export` 语法）和 CommonJS 模块（`require()`/`module.exports`）。以下 CommonJS 版本在 Bun 中也可以工作。

{% codetabs %}

```ts#index.js
const { hello } = require("./hello");

hello();
```

```ts#hello.js
function hello() {
  console.log("Hello world!");
}

exports.hello = hello;
```

{% /codetabs %}

话虽如此，在新项目中不鼓励使用 CommonJS。

## 模块系统

Bun 原生支持 CommonJS 和 ES 模块。ES 模块是新项目推荐的模块格式，但 CommonJS 模块在 Node.js 生态系统中仍然广泛使用。

在 Bun 的 JavaScript 运行时中，`require` 可以被 ES 模块和 CommonJS 模块使用。如果目标模块是 ES 模块，`require` 返回模块命名空间对象（相当于 `import * as`）。如果目标模块是 CommonJS 模块，`require` 返回 `module.exports` 对象（与 Node.js 中一样）。

| 模块类型 | `require()`      | `import * as`                                                           |
| -------- | ---------------- | ----------------------------------------------------------------------- |
| ES 模块  | 模块命名空间     | 模块命名空间                                                            |
| CommonJS | module.exports   | `default` 是 `module.exports`，module.exports 的键是命名导出 |

### 使用 `require()`

您可以 `require()` 任何文件或包，甚至是 `.ts` 或 `.mjs` 文件。

```ts
const { foo } = require("./foo"); // 扩展名是可选的
const { bar } = require("./bar.mjs");
const { baz } = require("./baz.tsx");
```

{% details summary="什么是 CommonJS 模块？" %}

2016 年，ECMAScript 添加了对 [ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)的支持。ES 模块是 JavaScript 模块的标准。然而，数百万 npm 包仍然使用 CommonJS 模块。

CommonJS 模块是使用 `module.exports` 导出值的模块。通常，`require` 用于导入 CommonJS 模块。

```ts
// my-commonjs.cjs
const stuff = require("./stuff");
module.exports = { stuff };
```

CommonJS 和 ES 模块之间最大的区别是 CommonJS 模块是同步的，而 ES 模块是异步的。还有其他区别。

- ES 模块支持顶层 `await`，而 CommonJS 模块不支持。
- ES 模块始终处于[严格模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)，而 CommonJS 模块不是。
- 浏览器对 CommonJS 模块没有原生支持，但它们通过 `<script type="module">` 对 ES 模块有原生支持。
- CommonJS 模块不是静态可分析的，而 ES 模块只允许静态导入和导出。

**CommonJS 模块：** 这些是 JavaScript 中使用的一种模块系统。CommonJS 模块的一个关键特性是它们同步加载和执行。这意味着当您导入 CommonJS 模块时，该模块中的代码会立即运行，您的程序会等待它完成后再继续下一个任务。这就像从头到尾阅读一本书，不跳过任何页面。

**ES 模块 (ESM)：** 这些是 JavaScript 中引入的另一种模块系统。与 CommonJS 相比，它们的行为略有不同。在 ESM 中，静态导入（使用 `import` 语句进行的导入）是同步的，就像 CommonJS 一样。这意味着当您使用常规 `import` 语句导入 ESM 时，该模块中的代码会立即运行，您的程序会逐步进行。这就像一页一页地阅读一本书。

**动态导入：** 现在，这里有一个可能令人困惑的部分。ES 模块还支持通过 `import()` 函数动态导入模块。这被称为"动态导入"，它是异步的，所以它不会阻塞主程序执行。相反，它在后台获取和加载模块，同时您的程序继续运行。一旦模块准备就绪，您就可以使用它。这就像在阅读一本书时获取额外信息，而不必暂停阅读。

**总结：**

- CommonJS 模块和静态 ES 模块（`import` 语句）以类似的方式同步工作，就像从头到尾阅读一本书。
- ES 模块还提供使用 `import()` 函数异步导入模块的选项。这就像在阅读过程中查找额外信息而不停止阅读。

{% /details %}

### 使用 `import`

您可以 `import` 任何文件或包，甚至是 `.cjs` 文件。

```ts
import { foo } from "./foo"; // 扩展名是可选的
import bar from "./bar.ts";
import { stuff } from "./my-commonjs.cjs";
```

### 一起使用 `import` 和 `require()`

在 Bun 中，您可以在同一个文件中使用 `import` 或 `require` — 它们都可以工作，随时都可以。

```ts
import { stuff } from "./my-commonjs.cjs";
import Stuff from "./my-commonjs.cjs";
const myStuff = require("./my-commonjs.cjs");
```

### 顶层 await

这条规则唯一的例外是顶层 await。您不能 `require()` 使用顶层 await 的文件，因为 `require()` 函数本质上是同步的。

幸运的是，很少有库使用顶层 await，所以这很少成为问题。但如果您在应用程序代码中使用顶层 await，请确保该文件不会被应用程序的其他部分 `require()`。相反，您应该使用 `import` 或[动态 `import()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import)。

## 导入包

Bun 实现了 Node.js 模块解析算法，所以您可以使用裸说明符从 `node_modules` 导入包。

```ts
import { stuff } from "foo";
```

这个算法的完整规范在 [Node.js 文档](https://nodejs.org/api/modules.html)中正式记录；我们不会在这里重复。简而言之：如果您从 `"foo"` 导入，Bun 会扫描文件系统寻找包含 `foo` 包的 `node_modules` 目录。

一旦找到 `foo` 包，Bun 读取 `package.json` 以确定应该如何导入该包。要确定包的入口点，Bun 首先读取 `exports` 字段并检查以下条件。

```jsonc#package.json
{
  "name": "foo",
  "exports": {
    "bun": "./index.js",
    "node": "./index.js",
    "require": "./index.js", // 如果导入者是 CommonJS
    "import": "./index.mjs", // 如果导入者是 ES 模块
    "default": "./index.js",
  }
}
```

在 `package.json` 中首先出现的这些条件之一用于确定包的入口点。

Bun 尊重子路径 [`"exports"`](https://nodejs.org/api/packages.html#subpath-exports) 和 [`"imports"`](https://nodejs.org/api/packages.html#imports)。

```jsonc#package.json
{
  "name": "foo",
  "exports": {
    ".": "./index.js"
  }
}
```

子路径导入和条件导入可以相互结合使用。

```json
{
  "name": "foo",
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js"
    }
  }
}
```

与 Node.js 一样，在 `"exports"` 映射中指定任何子路径将阻止其他子路径被导入；您只能导入明确导出的文件。给定上面的 `package.json`：

```ts
import stuff from "foo"; // 这可以工作
import stuff from "foo/index.mjs"; // 这不行
```

{% callout %}
**发布 TypeScript** — 注意 Bun 支持特殊的 `"bun"` 导出条件。如果您的库是用 TypeScript 编写的，您可以直接将（未转译的！）TypeScript 文件发布到 `npm`。如果您在 `"bun"` 条件中指定包的 `*.ts` 入口点，Bun 将直接导入并执行您的 TypeScript 源文件。
{% /callout %}

如果未定义 `exports`，Bun 回退到 `"module"`（仅 ESM 导入）然后是 [`"main"`](https://nodejs.org/api/packages.html#main)。

```json#package.json
{
  "name": "foo",
  "module": "./index.js",
  "main": "./index.js"
}
```

### 自定义条件

`--conditions` 标志允许您指定在从 package.json `"exports"` 解析包时要使用的条件列表。

这个标志在 `bun build` 和 Bun 的运行时中都受支持。

```sh
# 在 bun build 中使用它：
$ bun build --conditions="react-server" --target=bun ./app/foo/route.js

# 在 bun 的运行时中使用它：
$ bun --conditions="react-server" ./app/foo/route.js
```

您还可以使用 `conditions` 以编程方式与 `Bun.build` 一起使用：

```js
await Bun.build({
  conditions: ["react-server"],
  target: "bun",
  entryPoints: ["./app/foo/route.js"],
});
```

## 路径重映射

本着将 TypeScript 作为一等公民的精神，Bun 运行时将根据 `tsconfig.json` 中的 [`compilerOptions.paths`](https://www.typescriptlang.org/tsconfig#paths) 字段重映射导入路径。这与 Node.js 有很大的不同，Node.js 不支持任何形式的导入路径重映射。

```jsonc#tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "config": ["./config.ts"],         // 将说明符映射到文件
      "components/*": ["components/*"],  // 通配符匹配
    }
  }
}
```

如果您不是 TypeScript 用户，您可以在项目根目录创建一个 [`jsconfig.json`](https://code.visualstudio.com/docs/languages/jsconfig) 来实现相同的行为。

{% details summary="Bun 中 CommonJS 互操作的低级细节" %}

Bun 的 JavaScript 运行时原生支持 CommonJS。当 Bun 的 JavaScript 转译器检测到 `module.exports` 的使用时，它将文件视为 CommonJS。然后，模块加载器将转译后的模块包装在一个形状如下的函数中：

```js
(function (module, exports, require) {
  // 转译后的模块
})(module, exports, require);
```

`module`、`exports` 和 `require` 非常类似于 Node.js 中的 `module`、`exports` 和 `require`。这些是通过 C++ 中的 [`with scope`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/with) 分配的。内部 `Map` 存储 `exports` 对象以在模块完全加载之前处理循环 `require` 调用。

一旦 CommonJS 模块成功评估，就会创建一个合成模块记录，其中 ES 模块的 `default` [导出设置为 `module.exports`](https://github.com/oven-sh/bun/blob/9b6913e1a674ceb7f670f917fc355bb8758c6c72/src/bun.js/bindings/CommonJSModuleRecord.cpp#L212-L213)，并且 `module.exports` 对象的键被重新导出为命名导出（如果 `module.exports` 对象是一个对象）。

当使用 Bun 的打包器时，这工作方式不同。打包器将 CommonJS 模块包装在一个 `require_${moduleName}` 函数中，该函数返回 `module.exports` 对象。

{% /details %}
