Bun 每天都在向 100% Node.js API 兼容性迈进。如今，像 Next.js、Express 这样的流行框架以及数百万个为 Node 设计的 `npm` 包都可以直接在 Bun 中使用。为了确保兼容性，我们在每次发布 Bun 之前都会运行数千个来自 Node.js 测试套件的测试。

**如果一个包在 Node.js 中可以工作但在 Bun 中不能工作，我们认为这是 Bun 的一个 bug。**请[提交一个 issue](https://bun.sh/issues)，我们会修复它。

本页面会定期更新以反映最新版本 Bun 的兼容性状态。以下信息反映了 Bun 与 _Node.js v23_ 的兼容性。

## 内置 Node.js 模块

### [`node:assert`](https://nodejs.org/api/assert.html)

🟢 完全实现。

### [`node:buffer`](https://nodejs.org/api/buffer.html)

🟢 完全实现。

### [`node:console`](https://nodejs.org/api/console.html)

🟢 完全实现。

### [`node:dgram`](https://nodejs.org/api/dgram.html)

🟢 完全实现。> 90% 的 Node.js 测试套件通过。

### [`node:diagnostics_channel`](https://nodejs.org/api/diagnostics_channel.html)

🟢 完全实现。

### [`node:dns`](https://nodejs.org/api/dns.html)

🟢 完全实现。> 90% 的 Node.js 测试套件通过。

### [`node:events`](https://nodejs.org/api/events.html)

🟢 完全实现。100% 的 Node.js 测试套件通过。`EventEmitterAsyncResource` 在底层使用 `AsyncResource`。

### [`node:fs`](https://nodejs.org/api/fs.html)

🟢 完全实现。92% 的 Node.js 测试套件通过。

### [`node:http`](https://nodejs.org/api/http.html)

🟢 完全实现。目前传出客户端请求体是缓冲的而不是流式的。

### [`node:https`](https://nodejs.org/api/https.html)

🟢 API 已实现，但 `Agent` 尚未完全使用。

### [`node:os`](https://nodejs.org/api/os.html)

🟢 完全实现。100% 的 Node.js 测试套件通过。

### [`node:path`](https://nodejs.org/api/path.html)

🟢 完全实现。100% 的 Node.js 测试套件通过。

### [`node:punycode`](https://nodejs.org/api/punycode.html)

🟢 完全实现。100% 的 Node.js 测试套件通过，_已被 Node.js 弃用_。

### [`node:querystring`](https://nodejs.org/api/querystring.html)

🟢 完全实现。100% 的 Node.js 测试套件通过。

### [`node:readline`](https://nodejs.org/api/readline.html)

🟢 完全实现。

### [`node:stream`](https://nodejs.org/api/stream.html)

🟢 完全实现。

### [`node:string_decoder`](https://nodejs.org/api/string_decoder.html)

🟢 完全实现。100% 的 Node.js 测试套件通过。

### [`node:timers`](https://nodejs.org/api/timers.html)

🟢 建议使用全局 `setTimeout` 等替代。

### [`node:tty`](https://nodejs.org/api/tty.html)

🟢 完全实现。

### [`node:url`](https://nodejs.org/api/url.html)

🟢 完全实现。

### [`node:zlib`](https://nodejs.org/api/zlib.html)

🟢 完全实现。98% 的 Node.js 测试套件通过。

### [`node:async_hooks`](https://nodejs.org/api/async_hooks.html)

🟡 已实现 `AsyncLocalStorage` 和 `AsyncResource`。v8 promise hooks 未被调用，并且其使用[强烈不推荐](https://nodejs.org/docs/latest/api/async_hooks.html#async-hooks)。

### [`node:child_process`](https://nodejs.org/api/child_process.html)

🟡 缺少 `proc.gid` `proc.uid`。`Stream` 类未导出。IPC 不能发送套接字句柄。Node.js <> Bun IPC 可以使用 JSON 序列化。

### [`node:cluster`](https://nodejs.org/api/cluster.html)

🟡 句柄和文件描述符不能在 worker 之间传递，这意味着目前只在 Linux 上支持跨进程负载均衡 HTTP 请求（通过 `SO_REUSEPORT`）。否则，已实现但未经实战测试。

### [`node:crypto`](https://nodejs.org/api/crypto.html)

🟡 缺少 `secureHeapUsed` `setEngine` `setFips`

### [`node:domain`](https://nodejs.org/api/domain.html)

🟡 缺少 `Domain` `active`

### [`node:http2`](https://nodejs.org/api/http2.html)

🟡 客户端和服务器已实现（95.25% 的 gRPC 测试套件通过）。缺少 `options.allowHTTP1`、`options.enableConnectProtocol`、ALTSVC 扩展和 `http2stream.pushStream`。

### [`node:module`](https://nodejs.org/api/module.html)

🟡 缺少 `syncBuiltinESMExports`、`Module#load()`。支持为 ESM 和 CJS 模块覆盖 `require.cache`。`module._extensions`、`module._pathCache`、`module._cache` 是无操作。`module.register` 未实现，我们建议在此期间使用 [`Bun.plugin`](https://bun.sh/docs/runtime/plugins)。

### [`node:net`](https://nodejs.org/api/net.html)

🟡 `SocketAddress` 类未暴露（但已实现）。`BlockList` 存在但是无操作。

### [`node:perf_hooks`](https://nodejs.org/api/perf_hooks.html)

🟡 缺少 `createHistogram` `monitorEventLoopDelay`。建议使用 `performance` 全局变量而不是 `perf_hooks.performance`。

### [`node:process`](https://nodejs.org/api/process.html)

🟡 参见 [`process`](#process) 全局变量。

### [`node:sys`](https://nodejs.org/api/util.html)

🟡 参见 [`node:util`](#node-util)。

### [`node:tls`](https://nodejs.org/api/tls.html)

🟡 缺少 `tls.createSecurePair`。

### [`node:util`](https://nodejs.org/api/util.html)

🟡 缺少 `getCallSite` `getCallSites` `getSystemErrorMap` `getSystemErrorMessage` `transferableAbortSignal` `transferableAbortController`

### [`node:v8`](https://nodejs.org/api/v8.html)

🟡 已实现 `writeHeapSnapshot` 和 `getHeapSnapshot`。`serialize` 和 `deserialize` 使用 JavaScriptCore 的线格式而不是 V8 的。其他方法未实现。对于性能分析，请使用 [`bun:jsc`](https://bun.sh/docs/project/benchmarking#bunjsc) 替代。

### [`node:vm`](https://nodejs.org/api/vm.html)

🟡 核心功能可用，但实验性 VM ES 模块未实现，包括 `vm.Module`、`vm.SourceTextModule`、`vm.SyntheticModule`、`importModuleDynamically` 和 `vm.measureMemory`。`timeout`、`breakOnSigint`、`cachedData` 等选项尚未实现。

### [`node:wasi`](https://nodejs.org/api/wasi.html)

🟡 部分实现。

### [`node:worker_threads`](https://nodejs.org/api/worker_threads.html)

🟡 `Worker` 不支持以下选项：`stdin` `stdout` `stderr` `trackedUnmanagedFds` `resourceLimits`。缺少 `markAsUntransferable` `moveMessagePortToContext` `getHeapSnapshot`。

### [`node:inspector`](https://nodejs.org/api/inspector.html)

🔴 未实现。

### [`node:repl`](https://nodejs.org/api/repl.html)

🔴 未实现。

### [`node:sqlite`](https://nodejs.org/api/sqlite.html)

🔴 未实现。

### [`node:test`](https://nodejs.org/api/test.html)

🟡 部分实现。缺少模拟、快照、计时器。请使用 [`bun:test`](https://bun.sh/docs/cli/test) 替代。

### [`node:trace_events`](https://nodejs.org/api/tracing.html)

🔴 未实现。

## Node.js 全局变量

下表列出了 Node.js 实现的所有全局变量以及 Bun 当前的兼容性状态。

### [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)

🟢 完全实现。

### [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)

🟢 完全实现。

### [`Blob`](https://developer.mozilla.org/en-US/docs/Web/API/Blob)

🟢 完全实现。

### [`Buffer`](https://nodejs.org/api/buffer.html#class-buffer)

🟢 完全实现。

### [`ByteLengthQueuingStrategy`](https://developer.mozilla.org/en-US/docs/Web/API/ByteLengthQueuingStrategy)

🟢 完全实现。

### [`__dirname`](https://nodejs.org/api/globals.html#__dirname)

🟢 完全实现。

### [`__filename`](https://nodejs.org/api/globals.html#__filename)

🟢 完全实现。

### [`atob()`](https://developer.mozilla.org/en-US/docs/Web/API/atob)

🟢 完全实现。

### [`BroadcastChannel`](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)

🟢 完全实现。

### [`btoa()`](https://developer.mozilla.org/en-US/docs/Web/API/btoa)

🟢 完全实现。

### [`clearImmediate()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/clearImmediate)

🟢 完全实现。

### [`clearInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/clearInterval)

🟢 完全实现。

### [`clearTimeout()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/clearTimeout)

🟢 完全实现。

### [`CompressionStream`](https://developer.mozilla.org/en-US/docs/Web/API/CompressionStream)

🔴 未实现。

### [`console`](https://developer.mozilla.org/en-US/docs/Web/API/console)

🟢 完全实现。

### [`CountQueuingStrategy`](https://developer.mozilla.org/en-US/docs/Web/API/CountQueuingStrategy)

🟢 完全实现。

### [`Crypto`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto)

🟢 完全实现。

### [`SubtleCrypto (crypto)`](https://developer.mozilla.org/en-US/docs/Web/API/crypto)

🟢 完全实现。

### [`CryptoKey`](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey)

🟢 完全实现。

### [`CustomEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)

🟢 完全实现。

### [`DecompressionStream`](https://developer.mozilla.org/en-US/docs/Web/API/DecompressionStream)

🔴 未实现。

### [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event)

🟢 完全实现。

### [`EventTarget`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget)

🟢 完全实现。

### [`exports`](https://nodejs.org/api/globals.html#exports)

🟢 完全实现。

### [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/fetch)

🟢 完全实现。

### [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData)

🟢 完全实现。

### [`global`](https://nodejs.org/api/globals.html#global)

🟢 已实现。这是一个包含全局命名空间中所有对象的对象。很少直接引用它，因为它的内容可以直接使用，无需额外前缀，例如使用 `__dirname` 而不是 `global.__dirname`。

### [`globalThis`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis)

🟢 别名到 `global`。

### [`Headers`](https://developer.mozilla.org/en-US/docs/Web/API/Headers)

🟢 完全实现。

### [`MessageChannel`](https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel)

🟢 完全实现。

### [`MessageEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent)

🟢 完全实现。

### [`MessagePort`](https://developer.mozilla.org/en-US/docs/Web/API/MessagePort)

🟢 完全实现。

### [`module`](https://nodejs.org/api/globals.html#module)

🟢 完全实现。

### [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry)

🟢 完全实现。

### [`PerformanceMark`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceMark)

🟢 完全实现。

### [`PerformanceMeasure`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceMeasure)

🟢 完全实现。

### [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver)

🟢 完全实现。

### [`PerformanceObserverEntryList`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserverEntryList)

🟢 完全实现。

### [`PerformanceResourceTiming`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming)

🟢 完全实现。

### [`performance`](https://developer.mozilla.org/en-US/docs/Web/API/performance)

🟢 完全实现。

### [`process`](https://nodejs.org/api/process.html)

🟡 大部分已实现。`process.binding`（一些包依赖的内部 Node.js 绑定）部分实现。`process.title` 目前在 macOS 和 Linux 上是无操作。`getActiveResourcesInfo` `setActiveResourcesInfo`、`getActiveResources` 和 `setSourceMapsEnabled` 是存根。较新的 API 如 `process.loadEnvFile` 和 `process.getBuiltinModule` 尚未实现。

### [`queueMicrotask()`](https://developer.mozilla.org/en-US/docs/Web/API/queueMicrotask)

🟢 完全实现。

### [`ReadableByteStreamController`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableByteStreamController)

🟢 完全实现。

### [`ReadableStream`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

🟢 完全实现。

### [`ReadableStreamBYOBReader`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStreamBYOBReader)

🟢 完全实现。

### [`ReadableStreamBYOBRequest`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStreamBYOBRequest)

🟢 完全实现。

### [`ReadableStreamDefaultController`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStreamDefaultController)

🟢 完全实现。

### [`ReadableStreamDefaultReader`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStreamDefaultReader)

🟢 完全实现。

### [`require()`](https://nodejs.org/api/globals.html#require)

🟢 完全实现，包括 [`require.main`](https://nodejs.org/api/modules.html#requiremain)、[`require.cache`](https://nodejs.org/api/modules.html#requirecache)、[`require.resolve`](https://nodejs.org/api/modules.html#requireresolverequest-options)。

### [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response)

🟢 完全实现。

### [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request)

🟢 完全实现。

### [`setImmediate()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setImmediate)

🟢 完全实现。

### [`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setInterval)

🟢 完全实现。

### [`setTimeout()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout)

🟢 完全实现。

### [`structuredClone()`](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)

🟢 完全实现。

### [`SubtleCrypto`](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto)

🟢 完全实现。

### [`DOMException`](https://developer.mozilla.org/en-US/docs/Web/API/DOMException)

🟢 完全实现。

### [`TextDecoder`](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder)

🟢 完全实现。

### [`TextDecoderStream`](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream)

🟢 完全实现。

### [`TextEncoder`](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoder)

🟢 完全实现。

### [`TextEncoderStream`](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoderStream)

🟢 完全实现。

### [`TransformStream`](https://developer.mozilla.org/en-US/docs/Web/API/TransformStream)

🟢 完全实现。

### [`TransformStreamDefaultController`](https://developer.mozilla.org/en-US/docs/Web/API/TransformStreamDefaultController)

🟢 完全实现。

### [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL)

🟢 完全实现。

### [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

🟢 完全实现。

### [`WebAssembly`](https://nodejs.org/api/globals.html#webassembly)

🟢 完全实现。

### [`WritableStream`](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream)

🟢 完全实现。

### [`WritableStreamDefaultController`](https://developer.mozilla.org/en-US/docs/Web/API/WritableStreamDefaultController)

🟢 完全实现。

### [`WritableStreamDefaultWriter`](https://developer.mozilla.org/en-US/docs/Web/API/WritableStreamDefaultWriter)

🟢 完全实现。
