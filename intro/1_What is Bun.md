Bun 是一个用于 JavaScript 和 TypeScript 应用程序的一体化工具包。它以单个可执行文件 `bun` 的形式发布。

其核心是 _Bun 运行时_，这是一个快速的 JavaScript 运行时，设计为 **Node.js 的直接替代品**。它使用 Zig 编写，底层由 JavaScriptCore 提供支持，显著减少了启动时间和内存使用。

```bash
$ bun run index.tsx  # 默认支持 TS 和 JSX
```

`bun` 命令行工具还实现了测试运行器、脚本运行器和与 Node.js 兼容的包管理器，所有这些都比现有工具快得多，并且可以在现有的 Node.js 项目中使用，几乎不需要更改。

```bash
$ bun run start                 # 运行 `start` 脚本
$ bun install <pkg>             # 安装一个包
$ bun build ./index.tsx         # 为浏览器打包项目
$ bun test                      # 运行测试
$ bunx cowsay 'Hello, world!'   # 执行一个包
```

通过下面的快速链接开始使用，或继续阅读以了解更多关于 Bun 的信息。

{% block className="gap-2 grid grid-flow-row grid-cols-1 md:grid-cols-2" %}
{% arrowbutton href="/docs/installation" text="安装 Bun" /%}
{% arrowbutton href="/docs/quickstart" text="快速入门" /%}
{% arrowbutton href="/docs/cli/install" text="安装包" /%}
{% arrowbutton href="/docs/cli/bun-create" text="使用项目模板" /%}
{% arrowbutton href="/docs/bundler" text="为生产环境打包代码" /%}
{% arrowbutton href="/docs/api/http" text="构建 HTTP 服务器" /%}
{% arrowbutton href="/docs/api/websockets" text="构建 WebSocket 服务器" /%}
{% arrowbutton href="/docs/api/file-io" text="读写文件" /%}
{% arrowbutton href="/docs/api/sqlite" text="运行 SQLite 查询" /%}
{% arrowbutton href="/docs/cli/test" text="编写和运行测试" /%}
{% /block %}

## 什么是运行时？

JavaScript（或更正式地说，ECMAScript）只是一种编程语言的_规范_。任何人都可以编写一个 JavaScript _引擎_，它可以接收有效的 JavaScript 程序并执行它。目前最流行的两个引擎是 V8（由 Google 开发）和 JavaScriptCore（由 Apple 开发）。两者都是开源的。

但是大多数 JavaScript 程序并不是在真空中运行的。它们需要一种方式来访问外部世界以执行有用的任务。这就是_运行时_的作用。它们实现了额外的 API，这些 API 随后可供它们执行的 JavaScript 程序使用。

### 浏览器

值得注意的是，浏览器附带 JavaScript 运行时，这些运行时实现了一组通过全局 `window` 对象暴露的 Web 特定 API。浏览器执行的任何 JavaScript 代码都可以使用这些 API 在当前网页的上下文中实现交互或动态行为。

### Node.js

类似地，Node.js 是一个可以在非浏览器环境（如服务器）中使用的 JavaScript 运行时。由 Node.js 执行的 JavaScript 程序除了内置模块外，还可以访问一组 Node.js 特定的[全局变量](https://nodejs.org/api/globals.html)，如 `Buffer`、`process` 和 `__dirname`，这些模块用于执行操作系统级任务，如读写文件（`node:fs`）和网络（`node:net`、`node:http`）。Node.js 还实现了基于 CommonJS 的模块系统和解析算法，这早于 JavaScript 的原生模块系统。

Bun 被设计为 Node.js 的更快、更精简、更现代的替代品。

## 设计目标

Bun 是从头开始设计的，考虑了当今的 JavaScript 生态系统。

- **速度**。Bun 进程启动速度[比 Node.js 快 4 倍](https://twitter.com/jarredsumner/status/1499225725492076544)（自己试试看！）
- **TypeScript & JSX 支持**。您可以直接执行 `.jsx`、`.ts` 和 `.tsx` 文件；Bun 的转译器在执行前将这些文件转换为普通 JavaScript。
- **ESM & CommonJS 兼容性**。世界正在向 ES 模块（ESM）发展，但 npm 上仍有数百万个包需要 CommonJS。Bun 推荐使用 ES 模块，但也支持 CommonJS。
- **Web 标准 API**。Bun 实现了标准的 Web API，如 `fetch`、`WebSocket` 和 `ReadableStream`。Bun 由 Apple 为 Safari 开发的 JavaScriptCore 引擎提供支持，因此一些 API 如 [`Headers`](https://developer.mozilla.org/en-US/docs/Web/API/Headers) 和 [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL) 直接使用 [Safari 的实现](https://github.com/oven-sh/bun/blob/HEAD/src/bun.js/bindings/webcore/JSFetchHeaders.cpp)。
- **Node.js 兼容性**。除了支持 Node 风格的模块解析外，Bun 还致力于与内置的 Node.js 全局变量（`process`、`Buffer`）和模块（`path`、`fs`、`http` 等）完全兼容。_这是一项正在进行的工作，尚未完成。_ 有关当前状态，请参阅[兼容性页面](https://bun.sh/docs/runtime/nodejs-apis)。

Bun 不仅仅是一个运行时。长期目标是成为一个用于使用 JavaScript/TypeScript 构建应用程序的完整基础设施工具包，包括包管理器、转译器、打包器、脚本运行器、测试运行器等。
