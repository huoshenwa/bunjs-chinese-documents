Bun 在 `Bun` 全局对象和多个内置模块上实现了一组原生 API。这些 API 经过高度优化，代表了实现一些常见功能的"Bun 原生"方式。

Bun 尽可能实现标准的 Web API。Bun 主要在服务器端任务（如文件 I/O 和启动 HTTP 服务器）没有标准的情况下引入新的 API。在这些情况下，Bun 的方法仍然建立在 `Blob`、`URL` 和 `Request` 等标准 API 之上。

```ts
Bun.serve({
  fetch(req: Request) {
    return new Response("Success!");
  },
});
```

点击右侧列中的链接跳转到相关文档。

{% table %}

- 主题
- API

---

- HTTP 服务器
- [`Bun.serve`](https://bun.sh/docs/api/http#bun-serve)

---

- 打包器
- [`Bun.build`](https://bun.sh/docs/bundler)

---

- 文件 I/O
- [`Bun.file`](https://bun.sh/docs/api/file-io#reading-files-bun-file)
  [`Bun.write`](https://bun.sh/docs/api/file-io#writing-files-bun-write)

---

- 子进程
- [`Bun.spawn`](https://bun.sh/docs/api/spawn#spawn-a-process-bun-spawn)
  [`Bun.spawnSync`](https://bun.sh/docs/api/spawn#blocking-api-bun-spawnsync)

---

- TCP
- [`Bun.listen`](https://bun.sh/docs/api/tcp#start-a-server-bun-listen)
  [`Bun.connect`](https://bun.sh/docs/api/tcp#start-a-server-bun-listen)

---

- 转译器
- [`Bun.Transpiler`](https://bun.sh/docs/api/transpiler)

---

- 路由
- [`Bun.FileSystemRouter`](https://bun.sh/docs/api/file-system-router)

---

- HTML 流转换
- [`HTMLRewriter`](https://bun.sh/docs/api/html-rewriter)

---

- 哈希
- [`Bun.hash`](https://bun.sh/docs/api/hashing#bun-hash)
  [`Bun.CryptoHasher`](https://bun.sh/docs/api/hashing#bun-cryptohasher)

---

- import.meta
- [`import.meta`](https://bun.sh/docs/api/import-meta)

---

<!-- - [DNS](https://bun.sh/docs/api/dns)
- `Bun.dns`

--- -->

- SQLite
- [`bun:sqlite`](https://bun.sh/docs/api/sqlite)

---

- FFI
- [`bun:ffi`](https://bun.sh/docs/api/ffi)

---

- 测试
- [`bun:test`](https://bun.sh/docs/cli/test)

---

- Node-API
- [`Node-API`](https://bun.sh/docs/api/node-api)

---

- Glob
- [`Bun.Glob`](https://bun.sh/docs/api/glob)

---

- 工具函数
- [`Bun.version`](https://bun.sh/docs/api/utils#bun-version)
  [`Bun.revision`](https://bun.sh/docs/api/utils#bun-revision)
  [`Bun.env`](https://bun.sh/docs/api/utils#bun-env)
  [`Bun.main`](https://bun.sh/docs/api/utils#bun-main)
  [`Bun.sleep()`](https://bun.sh/docs/api/utils#bun-sleep)
  [`Bun.sleepSync()`](https://bun.sh/docs/api/utils#bun-sleepsync)
  [`Bun.which()`](https://bun.sh/docs/api/utils#bun-which)
  [`Bun.peek()`](https://bun.sh/docs/api/utils#bun-peek)
  [`Bun.openInEditor()`](https://bun.sh/docs/api/utils#bun-openineditor)
  [`Bun.deepEquals()`](https://bun.sh/docs/api/utils#bun-deepequals)
  [`Bun.escapeHTML()`](https://bun.sh/docs/api/utils#bun-escapehtml)
  [`Bun.fileURLToPath()`](https://bun.sh/docs/api/utils#bun-fileurltopath)
  [`Bun.pathToFileURL()`](https://bun.sh/docs/api/utils#bun-pathtofileurl)
  [`Bun.gzipSync()`](https://bun.sh/docs/api/utils#bun-gzipsync)
  [`Bun.gunzipSync()`](https://bun.sh/docs/api/utils#bun-gunzipsync)
  [`Bun.deflateSync()`](https://bun.sh/docs/api/utils#bun-deflatesync)
  [`Bun.inflateSync()`](https://bun.sh/docs/api/utils#bun-inflatesync)
  [`Bun.inspect()`](https://bun.sh/docs/api/utils#bun-inspect)
  [`Bun.nanoseconds()`](https://bun.sh/docs/api/utils#bun-nanoseconds)
  [`Bun.readableStreamTo*()`](https://bun.sh/docs/api/utils#bun-readablestreamto)
  [`Bun.resolveSync()`](https://bun.sh/docs/api/utils#bun-resolvesync)

{% /table %}
