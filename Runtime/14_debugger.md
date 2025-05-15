---
name: 调试
---

Bun 支持 [WebKit Inspector Protocol](https://github.com/oven-sh/bun/blob/main/packages/bun-inspector-protocol/src/protocol/jsc/index.d.ts)，因此您可以使用交互式调试器来调试代码。为了演示，让我们看一个简单的 Web 服务器示例。

## 调试 JavaScript 和 TypeScript

```ts#server.ts
Bun.serve({
  fetch(req){
    console.log(req.url);
    return new Response("Hello, world!");
  }
})
```

### `--inspect`

要在使用 Bun 运行代码时启用调试，请使用 `--inspect` 标志。这会自动在可用端口上启动一个 WebSocket 服务器，可用于检查正在运行的 Bun 进程。

```sh
$ bun --inspect server.ts
------------------ Bun Inspector ------------------
监听地址：
  ws://localhost:6499/0tqxs9exrgrm

在浏览器中检查：
  https://debug.bun.sh/#localhost:6499/0tqxs9exrgrm
------------------ Bun Inspector ------------------
```

### `--inspect-brk`

`--inspect-brk` 标志的行为与 `--inspect` 相同，但它会在执行脚本的第一行自动注入一个断点。这对于调试快速运行并立即退出的脚本很有用。

### `--inspect-wait`

`--inspect-wait` 标志的行为与 `--inspect` 相同，但代码在调试器附加到运行进程之前不会执行。

### 为调试器设置端口或 URL

无论您使用哪个标志，您都可以选择指定端口号、URL 前缀或两者。

```sh
$ bun --inspect=4000 server.ts
$ bun --inspect=localhost:4000 server.ts
$ bun --inspect=localhost:4000/prefix server.ts
```

## 调试器

各种调试工具可以连接到这个服务器，提供交互式调试体验。

### `debug.bun.sh`

Bun 在 [debug.bun.sh](https://debug.bun.sh) 托管了一个基于 Web 的调试器。它是 WebKit 的 [Web Inspector Interface](https://webkit.org/web-inspector/web-inspector-interface/) 的修改版本，Safari 用户会觉得很熟悉。

在浏览器中打开提供的 `debug.bun.sh` URL 开始调试会话。从这个界面，您可以查看运行文件的源代码，查看和设置断点，并使用内置控制台执行代码。

{% image src="https://github.com/oven-sh/bun/assets/3084745/e6a976a8-80cc-4394-8925-539025cc025d" alt="Bun 调试器截图，控制台标签" /%}

让我们设置一个断点。导航到 Sources 标签；您应该看到之前的代码。点击行号 `3` 在我们的 `console.log(req.url)` 语句上设置断点。

{% image src="https://github.com/oven-sh/bun/assets/3084745/3b69c7e9-25ff-4f9d-acc4-caa736862935" alt="Bun 调试器截图" /%}

然后在您的 Web 浏览器中访问 [`http://localhost:3000`](http://localhost:3000)。这将向我们的 `localhost` Web 服务器发送 HTTP 请求。看起来页面没有加载。为什么？因为程序在我们之前设置的断点处暂停了执行。

注意 UI 是如何变化的。

{% image src="https://github.com/oven-sh/bun/assets/3084745/8b565e58-5445-4061-9bc4-f41090dfe769" alt="Bun 调试器截图" /%}

此时，我们可以做很多事情来检查当前的执行环境。我们可以使用底部的控制台在程序的上下文中运行任意代码，完全访问断点处作用域中的变量。

{% image src="https://github.com/oven-sh/bun/assets/3084745/f4312b76-48ba-4a7d-b3b6-6205968ac681" /%}

在 Sources 窗格的右侧，我们可以看到当前作用域中的所有局部变量，并深入查看它们的属性和方法。在这里，我们正在检查 `req` 变量。

{% image src="https://github.com/oven-sh/bun/assets/3084745/63b7f843-5180-489c-aa94-87c486e68646" /%}

在 Sources 窗格的左上角，我们可以控制程序的执行。

{% image src="https://github.com/oven-sh/bun/assets/3084745/41b76deb-7371-4461-9d5d-81b5a6d2f7a4" /%}

以下是控制流按钮功能的速查表：

- _继续脚本执行_ — 继续运行程序直到下一个断点或异常。
- _单步跳过_ — 程序将继续到下一行。
- _单步进入_ — 如果当前语句包含函数调用，调试器将"进入"被调用的函数。
- _单步跳出_ — 如果当前语句是函数调用，调试器将完成调用执行，然后"跳出"函数到调用它的位置。

{% image src="https://github-production-user-asset-6210df.s3.amazonaws.com/3084745/261510346-6a94441c-75d3-413a-99a7-efa62365f83d.png" /%}

### Visual Studio Code 调试器

Visual Studio Code 中提供了对调试 Bun 脚本的实验性支持。要使用它，您需要安装 [Bun VSCode 扩展](https://bun.sh/guides/runtime/vscode-debugger)。

## 调试网络请求

`BUN_CONFIG_VERBOSE_FETCH` 环境变量允许您自动记录使用 `fetch()` 或 `node:http` 发出的网络请求。

| 值     | 描述                     |
| ------ | ------------------------ |
| `curl` | 将请求打印为 `curl` 命令 |
| `true` | 打印请求和响应信息       |
| `false` | 不打印任何内容。默认值   |

### 将 fetch 和 node:http 请求打印为 curl 命令

Bun 还支持通过将环境变量 `BUN_CONFIG_VERBOSE_FETCH` 设置为 `curl` 来将 `fetch()` 和 `node:http` 网络请求打印为 `curl` 命令。

```ts
process.env.BUN_CONFIG_VERBOSE_FETCH = "curl";

await fetch("https://example.com", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ foo: "bar" }),
});
```

这将 `fetch` 请求打印为单行 `curl` 命令，让您可以复制粘贴到终端中以复制请求。

```sh
[fetch] $ curl --http1.1 "https://example.com/" -X POST -H "content-type: application/json" -H "Connection: keep-alive" -H "User-Agent: Bun/$BUN_LATEST_VERSION" -H "Accept: */*" -H "Host: example.com" -H "Accept-Encoding: gzip, deflate, br" --compressed -H "Content-Length: 13" --data-raw "{\"foo\":\"bar\"}"
[fetch] > HTTP/1.1 POST https://example.com/
[fetch] > content-type: application/json
[fetch] > Connection: keep-alive
[fetch] > User-Agent: Bun/$BUN_LATEST_VERSION
[fetch] > Accept: */*
[fetch] > Host: example.com
[fetch] > Accept-Encoding: gzip, deflate, br
[fetch] > Content-Length: 13

[fetch] < 200 OK
[fetch] < Accept-Ranges: bytes
[fetch] < Cache-Control: max-age=604800
[fetch] < Content-Type: text/html; charset=UTF-8
[fetch] < Date: Tue, 18 Jun 2024 05:12:07 GMT
[fetch] < Etag: "3147526947"
[fetch] < Expires: Tue, 25 Jun 2024 05:12:07 GMT
[fetch] < Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
[fetch] < Server: EOS (vny/044F)
[fetch] < Content-Length: 1256
```

带有 `[fetch] >` 的行是来自本地代码的请求，带有 `[fetch] <` 的行是来自远程服务器的响应。

`BUN_CONFIG_VERBOSE_FETCH` 环境变量在 `fetch()` 和 `node:http` 请求中都受支持，所以它应该可以正常工作。

要打印不带 `curl` 命令的内容，将 `BUN_CONFIG_VERBOSE_FETCH` 设置为 `true`。

```ts
process.env.BUN_CONFIG_VERBOSE_FETCH = "true";

await fetch("https://example.com", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ foo: "bar" }),
});
```

这将以下内容打印到控制台：

```sh
[fetch] > HTTP/1.1 POST https://example.com/
[fetch] > content-type: application/json
[fetch] > Connection: keep-alive
[fetch] > User-Agent: Bun/$BUN_LATEST_VERSION
[fetch] > Accept: */*
[fetch] > Host: example.com
[fetch] > Accept-Encoding: gzip, deflate, br
[fetch] > Content-Length: 13

[fetch] < 200 OK
[fetch] < Accept-Ranges: bytes
[fetch] < Cache-Control: max-age=604800
[fetch] < Content-Type: text/html; charset=UTF-8
[fetch] < Date: Tue, 18 Jun 2024 05:12:07 GMT
[fetch] < Etag: "3147526947"
[fetch] < Expires: Tue, 25 Jun 2024 05:12:07 GMT
[fetch] < Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
[fetch] < Server: EOS (vny/044F)
[fetch] < Content-Length: 1256
```

## 堆栈跟踪和源码映射

Bun 会转译每个文件，这听起来像是您在控制台中看到的堆栈跟踪会无用地指向转译后的输出。为了解决这个问题，Bun 会自动为每个转译的文件生成并提供源码映射文件。当您在控制台中看到堆栈跟踪时，您可以点击文件路径并跳转到原始源代码，即使它是用 TypeScript 或 JSX 编写的，或者应用了其他转换。

Bun 在运行时按需转译文件时以及使用 `bun build` 提前预编译文件时都会自动加载源码映射。

### 语法高亮的源代码预览

为了帮助调试，Bun 在发生未处理的异常或拒绝时会自动打印一小段源代码预览。您可以通过调用 `Bun.inspect(error)` 来模拟此行为：

```ts
// 创建一个错误
const err = new Error("Something went wrong");
console.log(Bun.inspect(err, { colors: true }));
```

这会打印发生错误的源代码的语法高亮预览，以及错误消息和堆栈跟踪。

```js
1 | // 创建一个错误
2 | const err = new Error("Something went wrong");
                ^
error: Something went wrong
      at file.js:2:13
```

### V8 堆栈跟踪

Bun 使用 JavaScriptCore 作为其引擎，但 Node.js 生态系统和 npm 的很大一部分都期望使用 V8。JavaScript 引擎在 `error.stack` 格式化方面有所不同。Bun 旨在成为 Node.js 的替代品，这意味着即使引擎不同，我们也需要确保堆栈跟踪尽可能相似。

这就是为什么当您在 Bun 中记录 `error.stack` 时，`error.stack` 的格式与 Node.js 的 V8 引擎相同。当您使用期望 V8 堆栈跟踪的库时，这特别有用。

#### V8 堆栈跟踪 API

Bun 实现了 [V8 Stack Trace API](https://v8.dev/docs/stack-trace-api)，这是一组允许您操作堆栈跟踪的函数。

##### Error.prepareStackTrace

`Error.prepareStackTrace` 函数是一个全局函数，允许您自定义堆栈跟踪输出。此函数使用错误对象和 `CallSite` 对象数组调用，并允许您返回自定义堆栈跟踪。

```ts
Error.prepareStackTrace = (err, stack) => {
  return stack.map(callSite => {
    return callSite.getFileName();
  });
};

const err = new Error("Something went wrong");
console.log(err.stack);
// [ "error.js" ]
```

`CallSite` 对象具有以下方法：

| 方法                     | 返回                                               |
| ------------------------ | -------------------------------------------------- |
| `getThis`                | 函数调用的 `this` 值                               |
| `getTypeName`            | `this` 的类型                                      |
| `getFunction`            | 函数对象                                           |
| `getFunctionName`        | 函数名称（字符串）                                 |
| `getMethodName`          | 方法名称（字符串）                                 |
| `getFileName`            | 文件名或 URL                                       |
| `getLineNumber`          | 行号                                               |
| `getColumnNumber`        | 列号                                               |
| `getEvalOrigin`          | `undefined`                                        |
| `getScriptNameOrSourceURL` | 源 URL                                           |
| `isToplevel`             | 如果函数在全局作用域中则返回 `true`                |
| `isEval`                 | 如果函数是 `eval` 调用则返回 `true`                |
| `isNative`               | 如果函数是原生函数则返回 `true`                    |
| `isConstructor`          | 如果函数是构造函数则返回 `true`                    |
| `isAsync`                | 如果函数是 `async` 则返回 `true`                   |
| `isPromiseAll`           | 尚未实现。                                         |
| `getPromiseIndex`        | 尚未实现。                                         |
| `toString`               | 返回调用站点的字符串表示                           |

在某些情况下，`Function` 对象可能已经被垃圾回收，所以其中一些方法可能返回 `undefined`。

##### Error.captureStackTrace(error, startFn)

`Error.captureStackTrace` 函数允许您在代码中的特定点捕获堆栈跟踪，而不是在抛出错误的地方。

当您有回调或异步代码使得难以确定错误来源时，这很有帮助。`Error.captureStackTrace` 的第二个参数是您希望堆栈跟踪开始的函数。

例如，下面的代码将使 `err.stack` 指向调用 `fn()` 的代码，即使错误是在 `myInner` 中抛出的。

```ts
const fn = () => {
  function myInner() {
    throw err;
  }

  try {
    myInner();
  } catch (err) {
    console.log(err.stack);
    console.log("");
    console.log("-- captureStackTrace --");
    console.log("");
    Error.captureStackTrace(err, fn);
    console.log(err.stack);
  }
};

fn();
```

这将记录以下内容：

```sh
Error: here!
    at myInner (file.js:4:15)
    at fn (file.js:8:5)
    at module code (file.js:17:1)
    at moduleEvaluation (native)
    at moduleEvaluation (native)
    at <anonymous> (native)

-- captureStackTrace --

Error: here!
    at module code (file.js:17:1)
    at moduleEvaluation (native)
    at moduleEvaluation (native)
    at <anonymous> (native)
```
