Bun 实现了 WHATWG `fetch` 标准，并添加了一些扩展以满足服务器端 JavaScript 的需求。

Bun 也实现了 `node:http`，但通常建议使用 `fetch`。

## 发送 HTTP 请求

要发送 HTTP 请求，使用 `fetch`

```ts
const response = await fetch("http://example.com");

console.log(response.status); // => 200

const text = await response.text(); // 或者 response.json(), response.formData() 等
```

`fetch` 也适用于 HTTPS URL。

```ts
const response = await fetch("https://example.com");
```

你也可以向 `fetch` 传递一个 [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) 对象。

```ts
const request = new Request("http://example.com", {
  method: "POST",
  body: "Hello, world!",
});

const response = await fetch(request);
```

### 发送 POST 请求

要发送 POST 请求，传递一个设置了 `method` 属性为 `"POST"` 的对象。

```ts
const response = await fetch("http://example.com", {
  method: "POST",
  body: "Hello, world!",
});
```

`body` 可以是字符串、`FormData` 对象、`ArrayBuffer`、`Blob` 等。有关更多信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#setting_a_body)。

### 代理请求

要代理请求，传递一个设置了 `proxy` 属性为 URL 的对象。

```ts
const response = await fetch("http://example.com", {
  proxy: "http://proxy.com",
});
```

### 自定义头部

要设置自定义头部，传递一个设置了 `headers` 属性为对象的对象。

```ts
const response = await fetch("http://example.com", {
  headers: {
    "X-Custom-Header": "value",
  },
});
```

你也可以使用 [Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers) 对象设置头部。

```ts
const headers = new Headers();
headers.append("X-Custom-Header", "value");

const response = await fetch("http://example.com", {
  headers,
});
```

### 响应体

要读取响应体，使用以下方法之一：

- `response.text(): Promise<string>`：返回一个解析为字符串的响应体的 Promise。
- `response.json(): Promise<any>`：返回一个解析为 JSON 对象的响应体的 Promise。
- `response.formData(): Promise<FormData>`：返回一个解析为 `FormData` 对象的响应体的 Promise。
- `response.bytes(): Promise<Uint8Array>`：返回一个解析为 `Uint8Array` 的响应体的 Promise。
- `response.arrayBuffer(): Promise<ArrayBuffer>`：返回一个解析为 `ArrayBuffer` 的响应体的 Promise。
- `response.blob(): Promise<Blob>`：返回一个解析为 `Blob` 的响应体的 Promise。

#### 流式响应体

你可以使用异步迭代器来流式处理响应体。

```ts
const response = await fetch("http://example.com");

for await (const chunk of response.body) {
  console.log(chunk);
}
```

你也可以更直接地访问 `ReadableStream` 对象。

```ts
const response = await fetch("http://example.com");

const stream = response.body;

const reader = stream.getReader();
const { value, done } = await reader.read();
```

### 流式请求体

你也可以使用 `ReadableStream` 在请求体中流式传输数据：

```ts
const stream = new ReadableStream({
  start(controller) {
    controller.enqueue("Hello");
    controller.enqueue(" ");
    controller.enqueue("World");
    controller.close();
  },
});

const response = await fetch("http://example.com", {
  method: "POST",
  body: stream,
});
```

当使用 HTTP(S) 流时：

- 数据直接流式传输到网络，无需在内存中缓冲整个主体
- 如果连接丢失，流将被取消
- 除非流有已知大小，否则不会自动设置 `Content-Length` 头部

当使用 S3 流时：

- 对于 PUT/POST 请求，Bun 自动使用分段上传
- 流被分块消费并并行上传
- 可以通过 S3 选项监控进度

### 带超时的 URL 获取

要获取带超时的 URL，使用 `AbortSignal.timeout`：

```ts
const response = await fetch("http://example.com", {
  signal: AbortSignal.timeout(1000),
});
```

#### 取消请求

要取消请求，使用 `AbortController`：

```ts
const controller = new AbortController();

const response = await fetch("http://example.com", {
  signal: controller.signal,
});

controller.abort();
```

### Unix 域套接字

要使用 Unix 域套接字获取 URL，使用 `unix: string` 选项：

```ts
const response = await fetch("https://hostname/a/path", {
  unix: "/var/run/path/to/unix.sock",
  method: "POST",
  body: JSON.stringify({ message: "Hello from Bun!" }),
  headers: {
    "Content-Type": "application/json",
  },
});
```

### TLS

要使用客户端证书，使用 `tls` 选项：

```ts
await fetch("https://example.com", {
  tls: {
    key: Bun.file("/path/to/key.pem"),
    cert: Bun.file("/path/to/cert.pem"),
    // ca: [Bun.file("/path/to/ca.pem")],
  },
});
```

#### 自定义 TLS 验证

要自定义 TLS 验证，在 `tls` 中使用 `checkServerIdentity` 选项

```ts
await fetch("https://example.com", {
  tls: {
    checkServerIdentity: (hostname, peerCertificate) => {
      // 如果证书无效，返回一个 Error
    },
  },
});
```

这与 Node 的 `net` 模块的工作方式类似。

#### 禁用 TLS 验证

要禁用 TLS 验证，将 `rejectUnauthorized` 设置为 `false`：

```ts
await fetch("https://example.com", {
  tls: {
    rejectUnauthorized: false,
  },
});
```

这在避免使用自签名证书时的 SSL 错误时特别有用，但这会禁用 TLS 验证，应谨慎使用。

### 请求选项

除了标准的 fetch 选项外，Bun 还提供了几个扩展：

```ts
const response = await fetch("http://example.com", {
  // 控制自动响应解压缩（默认：true）
  decompress: true,

  // 为此请求禁用连接重用
  keepalive: false,

  // 调试日志级别
  verbose: true, // 或 "curl" 获取更详细的输出
});
```

### 协议支持

除了 HTTP(S)，Bun 的 fetch 还支持几个额外的协议：

#### S3 URL - `s3://`

Bun 支持直接从 S3 存储桶获取。

```ts
// 使用环境变量作为凭证
const response = await fetch("s3://my-bucket/path/to/object");

// 或显式传递凭证
const response = await fetch("s3://my-bucket/path/to/object", {
  s3: {
    accessKeyId: "YOUR_ACCESS_KEY",
    secretAccessKey: "YOUR_SECRET_KEY",
    region: "us-east-1",
  },
});
```

注意：使用 S3 时，只有 PUT 和 POST 方法支持请求体。对于上传，Bun 自动对流式主体使用分段上传。

你可以在 [S3](https://bun.sh/docs/api/s3) 文档中了解更多关于 Bun 的 S3 支持。

#### 文件 URL - `file://`

你可以使用 `file:` 协议获取本地文件：

```ts
const response = await fetch("file:///path/to/file.txt");
const text = await response.text();
```

在 Windows 上，路径会自动规范化：

```ts
// 两者在 Windows 上都有效
const response = await fetch("file:///C:/path/to/file.txt");
const response2 = await fetch("file:///c:/path\\to/file.txt");
```

#### 数据 URL - `data:`

Bun 支持 `data:` URL 方案：

```ts
const response = await fetch("data:text/plain;base64,SGVsbG8sIFdvcmxkIQ==");
const text = await response.text(); // "Hello, World!"
```

#### Blob URL - `blob:`

你可以使用 `URL.createObjectURL()` 创建的 URL 获取 blob：

```ts
const blob = new Blob(["Hello, World!"], { type: "text/plain" });
const url = URL.createObjectURL(blob);
const response = await fetch(url);
```

### 错误处理

Bun 的 fetch 实现包括几个特定的错误情况：

- 在 GET/HEAD 方法中使用请求体会抛出错误（这是 fetch API 的预期行为）
- 同时使用 `proxy` 和 `unix` 选项会抛出错误
- 当 `rejectUnauthorized` 为 true（或未定义）时的 TLS 证书验证失败
- S3 操作可能会抛出与身份验证或权限相关的特定错误

### Content-Type 处理

当未明确提供时，Bun 会自动为请求体设置 `Content-Type` 头部：

- 对于 `Blob` 对象，使用 blob 的 `type`
- 对于 `FormData`，设置适当的多部分边界
- 对于 JSON 对象，设置 `application/json`

## 调试

为了帮助调试，你可以向 `fetch` 传递 `verbose: true`：

```ts
const response = await fetch("http://example.com", {
  verbose: true,
});
```

这将在终端打印请求和响应头部：

```sh
[fetch] > HTTP/1.1 GET http://example.com/
[fetch] > Connection: keep-alive
[fetch] > User-Agent: Bun/$BUN_LATEST_VERSION
[fetch] > Accept: */*
[fetch] > Host: example.com
[fetch] > Accept-Encoding: gzip, deflate, br

[fetch] < 200 OK
[fetch] < Content-Encoding: gzip
[fetch] < Age: 201555
[fetch] < Cache-Control: max-age=604800
[fetch] < Content-Type: text/html; charset=UTF-8
[fetch] < Date: Sun, 21 Jul 2024 02:41:14 GMT
[fetch] < Etag: "3147526947+gzip"
[fetch] < Expires: Sun, 28 Jul 2024 02:41:14 GMT
[fetch] < Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
[fetch] < Server: ECAcc (sac/254F)
[fetch] < Vary: Accept-Encoding
[fetch] < X-Cache: HIT
[fetch] < Content-Length: 648
```

注意：`verbose: boolean` 不是 Web 标准 `fetch` API 的一部分，是 Bun 特有的。

## 性能

在发送 HTTP 请求之前，必须执行 DNS 查找。这可能需要相当长的时间，特别是如果 DNS 服务器很慢或网络连接很差。

DNS 查找之后，必须连接 TCP 套接字，可能还需要执行 TLS 握手。这也可能需要相当长的时间。

请求完成后，消费响应体也可能需要相当长的时间和内存。

在每一步，Bun 都提供了 API 来帮助你优化应用程序的性能。

### DNS 预取

要预取 DNS 条目，你可以使用 `dns.prefetch` API。当你知道很快需要连接到主机并想要避免初始 DNS 查找时，这个 API 很有用。

```ts
import { dns } from "bun";

dns.prefetch("bun.sh");
```

#### DNS 缓存

默认情况下，Bun 在内存中缓存和去重 DNS 查询，最多 30 秒。你可以通过调用 `dns.getCacheStats()` 查看缓存统计信息：

要了解有关 Bun 中 DNS 缓存的更多信息，请参阅 [DNS 缓存](https://bun.sh/docs/api/dns) 文档。

### 预连接到主机

要预连接到主机，你可以使用 `fetch.preconnect` API。当你知道很快需要连接到主机并想要提前开始初始 DNS 查找、TCP 套接字连接和 TLS 握手时，这个 API 很有用。

```ts
import { fetch } from "bun";

fetch.preconnect("https://bun.sh");
```

注意：在 `fetch.preconnect` 之后立即调用 `fetch` 不会使你的请求更快。预连接只在你知道很快需要连接到主机但还没有准备好发出请求时才有帮助。

#### 在启动时预连接

要在启动时预连接到主机，你可以传递 `--fetch-preconnect`：

```sh
$ bun --fetch-preconnect https://bun.sh ./my-script.ts
```

这有点像 HTML 中的 `<link rel="preconnect">`。

此功能尚未在 Windows 上实现。如果你有兴趣在 Windows 上使用此功能，请提交一个 issue，我们可以为 Windows 实现支持。

### 连接池和 HTTP keep-alive

Bun 自动重用到同一主机的连接。这被称为连接池。这可以显著减少建立连接所需的时间。你不需要做任何事情来启用这个功能；它是自动的。

#### 同时连接限制

默认情况下，Bun 将同时 `fetch` 请求的最大数量限制为 256。我们这样做有几个原因：

- 它提高了整体系统稳定性。操作系统对同时打开的 TCP 套接字数量有上限，通常在几千个左右。接近这个限制会导致你的整个计算机行为异常。应用程序会挂起和崩溃。
- 它鼓励 HTTP Keep-Alive 连接重用。对于短命的 HTTP 请求，最慢的步骤通常是初始连接设置。重用连接可以节省大量时间。

当超过限制时，请求会被排队，并在下一个请求结束时发送。

你可以通过 `BUN_CONFIG_MAX_HTTP_REQUESTS` 环境变量增加最大同时连接数：

```sh
$ BUN_CONFIG_MAX_HTTP_REQUESTS=512 bun ./my-script.ts
```

此限制的最大值目前设置为 65,336。最大端口号是 65,535，所以任何一台计算机都很难超过这个限制。

### 响应缓冲

Bun 竭尽全力优化读取响应体的性能。读取响应体的最快方式是使用以下方法之一：

- `response.text(): Promise<string>`
- `response.json(): Promise<any>`
- `response.formData(): Promise<FormData>`
- `response.bytes(): Promise<Uint8Array>`
- `response.arrayBuffer(): Promise<ArrayBuffer>`
- `response.blob(): Promise<Blob>`

你也可以使用 `Bun.write` 将响应体写入磁盘上的文件：

```ts
import { write } from "bun";

await write("output.txt", response);
```

### 实现细节

- 默认启用连接池，但可以通过 `keepalive: false` 按请求禁用。也可以使用 `"Connection: close"` 头部禁用 keep-alive。
- 在特定条件下，使用操作系统的 `sendfile` 系统调用优化大文件上传：
  - 文件必须大于 32KB
  - 请求不能使用代理
  - 在 macOS 上，只有常规文件（不是管道、套接字或设备）可以使用 `sendfile`
  - 当不满足这些条件时，或使用 S3/流式上传时，Bun 会回退到将文件读入内存
  - 这种优化对于 HTTP（不是 HTTPS）请求特别有效，因为文件可以直接从内核发送到网络栈
- S3 操作自动处理请求签名和合并身份验证头部

注意：这些功能中有许多是 Bun 对标准 fetch API 的扩展。
