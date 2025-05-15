本页主要记录了 Bun 原生的 `Bun.serve` API。Bun 还实现了 [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 和 Node.js 的 [`http`](https://nodejs.org/api/http.html) 和 [`https`](https://nodejs.org/api/https.html) 模块。

{% callout %}
这些模块已经重新实现，以使用 Bun 的快速内部 HTTP 基础设施。你可以直接使用这些模块；依赖这些模块的框架（如 [Express](https://expressjs.com/)）应该可以直接工作。有关详细的兼容性信息，请参阅 [Runtime > Node.js APIs](https://bun.sh/docs/runtime/nodejs-apis)。
{% /callout %}

要启动一个具有简洁 API 的高性能 HTTP 服务器，推荐的方法是 [`Bun.serve`](#start-a-server-bun-serve)。

## `Bun.serve()`

使用 `Bun.serve` 在 Bun 中启动 HTTP 服务器。

```ts
Bun.serve({
  // `routes` 需要 Bun v1.2.3+
  routes: {
    // 静态路由
    "/api/status": new Response("OK"),

    // 动态路由
    "/users/:id": req => {
      return new Response(`Hello User ${req.params.id}!`);
    },

    // 每个 HTTP 方法的处理器
    "/api/posts": {
      GET: () => new Response("List posts"),
      POST: async req => {
        const body = await req.json();
        return Response.json({ created: true, ...body });
      },
    },

    // 通配符路由，匹配所有以 "/api/" 开头且未被其他路由匹配的路由
    "/api/*": Response.json({ message: "Not found" }, { status: 404 }),

    // 从 /blog/hello 重定向到 /blog/hello/world
    "/blog/hello": Response.redirect("/blog/hello/world"),

    // 通过缓冲在内存中来提供文件
    "/favicon.ico": new Response(await Bun.file("./favicon.ico").bytes(), {
      headers: {
        "Content-Type": "image/x-icon",
      },
    }),
  },

  // （可选）未匹配路由的回退：
  // 如果 Bun 版本 < 1.2.3，则需要
  fetch(req) {
    return new Response("Not Found", { status: 404 });
  },
});
```

### 路由

`Bun.serve()` 中的路由接收一个 `BunRequest`（它扩展了 [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request)）并返回一个 [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) 或 `Promise<Response>`。这使得在发送和接收 HTTP 请求时使用相同的代码更容易。

```ts
// 为简洁起见进行了简化
interface BunRequest<T extends string> extends Request {
  params: Record<T, string>;
  readonly cookies: CookieMap;
}
```

#### 路由中的异步/等待

你可以在路由处理器中使用 async/await 来返回 `Promise<Response>`。

```ts
import { sql, serve } from "bun";

serve({
  port: 3001,
  routes: {
    "/api/version": async () => {
      const [version] = await sql`SELECT version()`;
      return Response.json(version);
    },
  },
});
```

#### 路由中的 Promise

你也可以从路由处理器返回 `Promise<Response>`。

```ts
import { sql, serve } from "bun";

serve({
  routes: {
    "/api/version": () => {
      return new Promise(resolve => {
        setTimeout(async () => {
          const [version] = await sql`SELECT version()`;
          resolve(Response.json(version));
        }, 100);
      });
    },
  },
});
```

#### 类型安全的路由参数

当作为字符串字面量传递时，TypeScript 会解析路由参数，这样你的编辑器在访问 `request.params` 时会显示自动完成。

```ts
import type { BunRequest } from "bun";

Bun.serve({
  routes: {
    // 当作为字符串字面量传递时，TypeScript 知道 params 的形状
    "/orgs/:orgId/repos/:repoId": req => {
      const { orgId, repoId } = req.params;
      return Response.json({ orgId, repoId });
    },

    "/orgs/:orgId/repos/:repoId/settings": (
      // 可选：你可以显式地向 BunRequest 传递类型：
      req: BunRequest<"/orgs/:orgId/repos/:repoId/settings">,
    ) => {
      const { orgId, repoId } = req.params;
      return Response.json({ orgId, repoId });
    },
  },
});
```

路由参数值会自动进行百分比解码。支持 Unicode 字符。无效的 Unicode 会被替换为 Unicode 替换字符 `&0xFFFD;`。

### 静态响应

路由也可以是 `Response` 对象（没有处理器函数）。Bun.serve() 优化它以进行零分配调度 - 非常适合健康检查、重定向和固定内容：

```ts
Bun.serve({
  routes: {
    // 健康检查
    "/health": new Response("OK"),
    "/ready": new Response("Ready", {
      headers: {
        // 传递自定义头部
        "X-Ready": "1",
      },
    }),

    // 重定向
    "/blog": Response.redirect("https://bun.sh/blog"),

    // API 响应
    "/api/config": Response.json({
      version: "1.0.0",
      env: "production",
    }),
  },
});
```

静态响应在初始化后不会分配额外的内存。通常，你可以期望比手动返回 `Response` 对象至少提高 15% 的性能。

静态路由响应在服务器对象的生命周期内被缓存。要重新加载静态路由，调用 `server.reload(options)`。

```ts
const server = Bun.serve({
  static: {
    "/api/time": new Response(new Date().toISOString()),
  },

  fetch(req) {
    return new Response("404!");
  },
});

// 每秒更新时间。
setInterval(() => {
  server.reload({
    static: {
      "/api/time": new Response(new Date().toISOString()),
    },

    fetch(req) {
      return new Response("404!");
    },
  });
}, 1000);
```

重新加载路由只会影响下一个请求。正在进行的请求继续使用旧路由。在旧路由的正在进行的请求完成后，旧路由会从内存中释放。

为了简化错误处理，静态路由不支持从 `ReadableStream` 或 `AsyncIterator` 流式传输响应体。幸运的是，你可以先在内存中缓冲响应：

```ts
const time = await fetch("https://api.example.com/v1/data");
// 先在内存中缓冲响应。
const blob = await time.blob();

const server = Bun.serve({
  static: {
    "/api/data": new Response(blob),
  },

  fetch(req) {
    return new Response("404!");
  },
});
```

### 路由优先级

路由按特异性顺序匹配：

1. 精确路由（`/users/all`）
2. 参数路由（`/users/:id`）
3. 通配符路由（`/users/*`）
4. 全局捕获所有（`/*`）

```ts
Bun.serve({
  routes: {
    // 最具体的先来
    "/api/users/me": () => new Response("Current user"),
    "/api/users/:id": req => new Response(`User ${req.params.id}`),
    "/api/*": () => new Response("API catch-all"),
    "/*": () => new Response("Global catch-all"),
  },
});
```

### 每个 HTTP 方法的路由

路由处理器可以按 HTTP 方法专门化：

```ts
Bun.serve({
  routes: {
    "/api/posts": {
      // 每个方法的不同处理器
      GET: () => new Response("List posts"),
      POST: async req => {
        const post = await req.json();
        return Response.json({ id: crypto.randomUUID(), ...post });
      },
      PUT: async req => {
        const updates = await req.json();
        return Response.json({ updated: true, ...updates });
      },
      DELETE: () => new Response(null, { status: 204 }),
    },
  },
});
```

你可以传递以下任何方法：

| 方法     | 用例示例                 |
| -------- | ----------------------- |
| `GET`    | 获取资源                |
| `HEAD`   | 检查资源是否存在        |
| `OPTIONS`| 获取允许的 HTTP 方法（CORS）|
| `DELETE` | 删除资源                |
| `PATCH`  | 更新资源                |
| `POST`   | 创建资源                |
| `PUT`    | 更新资源                |

当传递函数而不是对象时，所有方法都将由该函数处理：

```ts
const server = Bun.serve({
  routes: {
    "/api/version": () => Response.json({ version: "1.0.0" }),
  },
});

await fetch(new URL("/api/version", server.url));
await fetch(new URL("/api/version", server.url), { method: "PUT" });
// ... 等等
```

### 热路由重载

使用 `server.reload()` 更新路由而无需重启服务器：

```ts
const server = Bun.serve({
  routes: {
    "/api/version": () => Response.json({ version: "1.0.0" }),
  },
});

// 无停机时间部署新路由
server.reload({
  routes: {
    "/api/version": () => Response.json({ version: "2.0.0" }),
  },
});
```

### 错误处理

Bun 为路由提供结构化错误处理：

```ts
Bun.serve({
  routes: {
    // 错误会被自动捕获
    "/api/risky": () => {
      throw new Error("Something went wrong");
    },
  },
  // 全局错误处理器
  error(error) {
    console.error(error);
    return new Response(`Internal Error: ${error.message}`, {
      status: 500,
      headers: {
        "Content-Type": "text/plain",
      },
    });
  },
});
```

### HTML 导入

要添加客户端单页应用，你可以使用 HTML 导入：

```ts
import myReactSinglePageApp from "./index.html";

Bun.serve({
  routes: {
    "/": myReactSinglePageApp,
  },
});
```

HTML 导入不仅仅是提供 HTML。它是一个使用 Bun 的[打包器](https://bun.sh/docs/bundler)、JavaScript 转译器和 CSS 解析器构建的全功能前端打包器、转译器和工具包。

你可以使用它来构建具有 React、TypeScript、Tailwind CSS 等的全功能前端。查看 [/docs/bundler/fullstack](https://bun.sh/docs/bundler/fullstack) 了解更多。

### 实际示例：REST API

这是一个使用 Bun 的路由器和零依赖的基本数据库支持的 REST API：

{% codetabs %}

```ts#server.ts
import type { Post } from "./types.ts";
import { Database } from "bun:sqlite";

const db = new Database("posts.db");
db.exec(`
  CREATE TABLE IF NOT EXISTS posts (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    created_at TEXT NOT NULL
  )
`);

Bun.serve({
  routes: {
    // 列出帖子
    "/api/posts": {
      GET: () => {
        const posts = db.query("SELECT * FROM posts").all();
        return Response.json(posts);
      },

      // 创建帖子
      POST: async req => {
        const post: Omit<Post, "id" | "created_at"> = await req.json();
        const id = crypto.randomUUID();

        db.query(
          `INSERT INTO posts (id, title, content, created_at)
           VALUES (?, ?, ?, ?)`,
        ).run(id, post.title, post.content, new Date().toISOString());

        return Response.json({ id, ...post }, { status: 201 });
      },
    },

    // 通过 ID 获取帖子
    "/api/posts/:id": req => {
      const post = db
        .query("SELECT * FROM posts WHERE id = ?")
        .get(req.params.id);

      if (!post) {
        return new Response("Not Found", { status: 404 });
      }

      return Response.json(post);
    },
  },

  error(error) {
    console.error(error);
    return new Response("Internal Server Error", { status: 500 });
  },
});
```

```ts#types.ts
export interface Post {
  id: string;
  title: string;
  content: string;
  created_at: string;
}
```

{% /codetabs %}

### 路由性能

`Bun.serve()` 的路由器建立在 uWebSocket 的[基于树的方法](https://github.com/oven-sh/bun/blob/0d1a00fa0f7830f8ecd99c027fce8096c9d459b6/packages/bun-uws/src/HttpRouter.h#L57-L64)之上，添加了 [SIMD 加速的路由参数解码](https://github.com/oven-sh/bun/blob/main/src/bun.js/bindings/decodeURIComponentSIMD.cpp#L21-L271) 和 [JavaScriptCore 结构缓存](https://github.com/oven-sh/bun/blob/main/src/bun.js/bindings/ServerRouteList.cpp#L100-L101)，以推动现代硬件允许的性能极限。

### `fetch` 请求处理器

`fetch` 处理器处理未被任何路由匹配的传入请求。它接收一个 [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) 对象并返回一个 [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) 或 [`Promise<Response>`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

```ts
Bun.serve({
  fetch(req) {
    const url = new URL(req.url);
    if (url.pathname === "/") return new Response("Home page!");
    if (url.pathname === "/blog") return new Response("Blog!");
    return new Response("404!");
  },
});
```

`fetch` 处理器支持 async/await：

```ts
import { sleep, serve } from "bun";
serve({
  async fetch(req) {
    const start = performance.now();
    await sleep(10);
    const end = performance.now();
    return new Response(`Slept for ${end - start}ms`);
  },
});
```

也支持基于 Promise 的响应：

```ts
Bun.serve({
  fetch(req) {
    // 将请求转发到另一个服务器。
    return fetch("https://example.com");
  },
});
```

你也可以从 `fetch` 处理器访问 `Server` 对象。它作为第二个参数传递给 `fetch` 函数。

```ts
// `server` 作为第二个参数传递给 `fetch`。
const server = Bun.serve({
  fetch(req, server) {
    const ip = server.requestIP(req);
    return new Response(`Your IP is ${ip}`);
  },
});
```

### 更改 `port` 和 `hostname`

要配置服务器将监听的端口和主机名，在选项对象中设置 `port` 和 `hostname`。

```ts
Bun.serve({
  port: 8080, // 默认为 $BUN_PORT, $PORT, $NODE_PORT，否则为 3000
  hostname: "mydomain.com", // 默认为 "0.0.0.0"
  fetch(req) {
    return new Response("404!");
  },
});
```

要随机选择一个可用端口，将 `port` 设置为 `0`。

```ts
const server = Bun.serve({
  port: 0, // 随机端口
  fetch(req) {
    return new Response("404!");
  },
});

// server.port 是随机选择的端口
console.log(server.port);
```

你可以通过访问服务器对象上的 `port` 属性或访问 `url` 属性来查看选择的端口。

```ts
console.log(server.port); // 3000
console.log(server.url); // http://localhost:3000
```

#### 配置默认端口

Bun 支持几个选项和环境变量来配置默认端口。当未设置 `port` 选项时使用默认端口。

- `--port` CLI 标志

```sh
$ bun --port=4002 server.ts
```

- `BUN_PORT` 环境变量

```sh
$ BUN_PORT=4002 bun server.ts
```

- `PORT` 环境变量

```sh
$ PORT=4002 bun server.ts
```

- `NODE_PORT` 环境变量

```sh
$ NODE_PORT=4002 bun server.ts
```

### Unix 域套接字

要监听 [unix 域套接字](https://en.wikipedia.org/wiki/Unix_domain_socket)，传递带有套接字路径的 `unix` 选项。

```ts
Bun.serve({
  unix: "/tmp/my-socket.sock", // 套接字路径
  fetch(req) {
    return new Response(`404!`);
  },
});
```

### 抽象命名空间套接字

Bun 支持 Linux 抽象命名空间套接字。要使用抽象命名空间套接字，在 `unix` 路径前加上空字节。

```ts
Bun.serve({
  unix: "\0my-abstract-socket", // 抽象命名空间套接字
  fetch(req) {
    return new Response(`404!`);
  },
});
```

与 unix 域套接字不同，抽象命名空间套接字不绑定到文件系统，并在关闭对套接字的最后一个引用时自动删除。

## 错误处理

要激活开发模式，设置 `development: true`。

```ts
Bun.serve({
  development: true,
  fetch(req) {
    throw new Error("woops!");
  },
});
```

在开发模式下，Bun 将在浏览器中显示带有内置错误页面的错误。

{% image src="/images/exception_page.png" caption="Bun 的内置 500 页面" /%}

### `error` 回调

要处理服务器端错误，实现一个 `error` 处理器。当发生错误时，此函数应返回要提供给客户端的 `Response`。在 `development` 模式下，此响应将取代 Bun 的默认错误页面。

```ts
Bun.serve({
  fetch(req) {
    throw new Error("woops!");
  },
  error(error) {
    return new Response(`<pre>${error}\n${error.stack}</pre>`, {
      headers: {
        "Content-Type": "text/html",
      },
    });
  },
});
```

{% callout %}
[了解有关 Bun 中调试的更多信息](https://bun.sh/docs/runtime/debugger)
{% /callout %}

对 `Bun.serve` 的调用返回一个 `Server` 对象。要停止服务器，调用 `.stop()` 方法。

```ts
const server = Bun.serve({
  fetch() {
    return new Response("Bun!");
  },
});

server.stop();
```

## TLS

Bun 开箱即用地支持 TLS，由 [BoringSSL](https://boringssl.googlesource.com/boringssl) 提供支持。通过为 `key` 和 `cert` 传递值来启用 TLS；启用 TLS 需要两者。

```ts-diff
  Bun.serve({
    fetch(req) {
      return new Response("Hello!!!");
    },

+   tls: {
+     key: Bun.file("./key.pem"),
+     cert: Bun.file("./cert.pem"),
+   }
  });
```

`key` 和 `cert` 字段期望你的 TLS 密钥和证书的_内容_，_而不是它的路径_。这可以是字符串、`BunFile`、`TypedArray` 或 `Buffer`。

```ts
Bun.serve({
  fetch() {},

  tls: {
    // BunFile
    key: Bun.file("./key.pem"),
    // Buffer
    key: fs.readFileSync("./key.pem"),
    // string
    key: fs.readFileSync("./key.pem", "utf8"),
    // 上面的数组
    key: [Bun.file("./key1.pem"), Bun.file("./key2.pem")],
  },
});
```

如果你的私钥使用密码短语加密，提供 `passphrase` 的值来解密它。

```ts-diff
  Bun.serve({
    fetch(req) {
      return new Response("Hello!!!");
    },

    tls: {
      key: Bun.file("./key.pem"),
      cert: Bun.file("./cert.pem"),
+     passphrase: "my-secret-passphrase",
    }
  });
```

可选地，你可以通过为 `ca` 传递值来覆盖受信任的 CA 证书。默认情况下，服务器将信任由 Mozilla 策划的知名 CA 列表。当指定 `ca` 时，Mozilla 列表将被覆盖。

```ts-diff
  Bun.serve({
    fetch(req) {
      return new Response("Hello!!!");
    },
    tls: {
      key: Bun.file("./key.pem"), // TLS 密钥路径
      cert: Bun.file("./cert.pem"), // TLS 证书路径
+     ca: Bun.file("./ca.pem"), // 根 CA 证书路径
    }
  });
```

要覆盖 Diffie-Hellman 参数：

```ts
Bun.serve({
  // ...
  tls: {
    // 其他配置
    dhParamsFile: "/path/to/dhparams.pem", // Diffie Hellman 参数路径
  },
});
```

### 服务器名称指示（SNI）

要配置服务器的服务器名称指示（SNI），在 `tls` 对象中设置 `serverName` 字段。

```ts
Bun.serve({
  // ...
  tls: {
    // ... 其他配置
    serverName: "my-server.com", // SNI
  },
});
```

要允许多个服务器名称，向 `tls` 传递一个对象数组，每个对象都有一个 `serverName` 字段。

```ts
Bun.serve({
  // ...
  tls: [
    {
      key: Bun.file("./key1.pem"),
      cert: Bun.file("./cert1.pem"),
      serverName: "my-server1.com",
    },
    {
      key: Bun.file("./key2.pem"),
      cert: Bun.file("./cert2.pem"),
      serverName: "my-server2.com",
    },
  ],
});
```

## idleTimeout

要配置空闲超时，在 Bun.serve 中设置 `idleTimeout` 字段。

```ts
Bun.serve({
  // 10 秒：
  idleTimeout: 10,

  fetch(req) {
    return new Response("Bun!");
  },
});
```

这是连接在服务器关闭它之前允许空闲的最大时间。如果没有发送或接收数据，则连接处于空闲状态。

## export default 语法

到目前为止，本页上的示例都使用了显式的 `Bun.serve` API。Bun 还支持替代语法。

```ts#server.ts
import {type Serve} from "bun";

export default {
  fetch(req) {
    return new Response("Bun!");
  },
} satisfies Serve;
```

不是将服务器选项传递给 `Bun.serve`，而是 `export default` 它。这个文件可以按原样执行；当 Bun 看到一个带有包含 `fetch` 处理器的 `default` 导出的文件时，它会在底层将其传递给 `Bun.serve`。

## 流式传输文件

要流式传输文件，返回一个以 `BunFile` 对象作为主体的 `Response` 对象。

```ts
Bun.serve({
  fetch(req) {
    return new Response(Bun.file("./hello.txt"));
  },
});
```

{% callout %}
⚡️ **速度** — 在可能的情况下，Bun 自动使用 [`sendfile(2)`](https://man7.org/linux/man-pages/man2/sendfile.2.html) 系统调用，在内核中启用零拷贝文件传输—发送文件的最快方式。
{% /callout %}

你可以使用 `Bun.file` 对象上的 [`slice(start, end)`](https://developer.mozilla.org/en-US/docs/Web/API/Blob/slice) 方法发送文件的一部分。这会自动在 `Response` 对象上设置 `Content-Range` 和 `Content-Length` 头部。

```ts
Bun.serve({
  fetch(req) {
    // 解析 `Range` 头部
    const [start = 0, end = Infinity] = req.headers
      .get("Range") // Range: bytes=0-100
      .split("=") // ["Range: bytes", "0-100"]
      .at(-1) // "0-100"
      .split("-") // ["0", "100"]
      .map(Number); // [0, 100]

    // 返回文件的一部分
    const bigFile = Bun.file("./big-video.mp4");
    return new Response(bigFile.slice(start, end));
  },
});
```

## 服务器生命周期方法

### server.stop() - 停止服务器

要停止服务器接受新连接：

```ts
const server = Bun.serve({
  fetch(req) {
    return new Response("Hello!");
  },
});

// 优雅地停止服务器（等待正在进行的请求）
await server.stop();

// 强制停止并关闭所有活动连接
await server.stop(true);
```

默认情况下，`stop()` 允许正在进行的请求和 WebSocket 连接完成。传递 `true` 以立即终止所有连接。

### server.ref() 和 server.unref() - 进程生命周期控制

控制服务器是否保持 Bun 进程存活：

```ts
// 如果服务器是唯一运行的东西，不要保持进程存活
server.unref();

// 恢复默认行为 - 保持进程存活
server.ref();
```

### server.reload() - 热重载处理器

更新服务器的处理器而无需重启：

```ts
const server = Bun.serve({
  routes: {
    "/api/version": Response.json({ version: "v1" }),
  },
  fetch(req) {
    return new Response("v1");
  },
});

// 更新到新的处理器
server.reload({
  routes: {
    "/api/version": Response.json({ version: "v2" }),
  },
  fetch(req) {
    return new Response("v2");
  },
});
```

这对于开发和热重载很有用。只能更新 `fetch`、`error` 和 `routes`。

## 每个请求的控制

### server.timeout(Request, seconds) - 自定义请求超时

为单个请求设置自定义空闲超时：

```ts
const server = Bun.serve({
  fetch(req, server) {
    // 为此请求设置 60 秒超时
    server.timeout(req, 60);

    // 如果他们发送主体需要超过 60 秒，请求将被中止
    await req.text();

    return new Response("Done!");
  },
});
```

传递 `0` 以禁用请求的超时。

### server.requestIP(Request) - 获取客户端信息

获取客户端 IP 和端口信息：

```ts
const server = Bun.serve({
  fetch(req, server) {
    const address = server.requestIP(req);
    if (address) {
      return new Response(
        `Client IP: ${address.address}, Port: ${address.port}`,
      );
    }
    return new Response("Unknown client");
  },
});
```

对于已关闭的请求或 Unix 域套接字返回 `null`。

## 使用 Cookie

Bun 提供了用于处理 HTTP 请求和响应中 cookie 的内置 API。`BunRequest` 对象包含一个 `cookies` 属性，它提供了一个 `CookieMap` 用于轻松访问和操作 cookie。使用 `routes` 时，`Bun.serve()` 自动跟踪 `request.cookies.set` 并将它们应用到响应。

### 读取 cookie

使用 `BunRequest` 对象上的 `cookies` 属性从传入请求中读取 cookie：

```ts
Bun.serve({
  routes: {
    "/profile": req => {
      // 从请求中访问 cookie
      const userId = req.cookies.get("user_id");
      const theme = req.cookies.get("theme") || "light";

      return Response.json({
        userId,
        theme,
        message: "Profile page",
      });
    },
  },
});
```

### 设置 cookie

要设置 cookie，使用 `BunRequest` 对象上的 `CookieMap` 的 `set` 方法。

```ts
Bun.serve({
  routes: {
    "/login": req => {
      const cookies = req.cookies;

      // 设置带有各种选项的 cookie
      cookies.set("user_id", "12345", {
        maxAge: 60 * 60 * 24 * 7, // 1 周
        httpOnly: true,
        secure: true,
        path: "/",
      });

      // 添加主题偏好 cookie
      cookies.set("theme", "dark");

      // 请求中修改的 cookie 会自动应用到响应
      return new Response("Login successful");
    },
  },
});
```

`Bun.serve()` 自动跟踪请求中修改的 cookie 并将它们应用到响应。

### 删除 cookie

要删除 cookie，使用 `request.cookies`（`CookieMap`）对象上的 `delete` 方法：

```ts
Bun.serve({
  routes: {
    "/logout": req => {
      // 删除 user_id cookie
      req.cookies.delete("user_id", {
        path: "/",
      });

      return new Response("Logged out successfully");
    },
  },
});
```

删除的 cookie 成为响应上的 `Set-Cookie` 头部，`maxAge` 设置为 `0`，`value` 为空。

## 服务器指标

### server.pendingRequests 和 server.pendingWebSockets

使用内置计数器监控服务器活动：

```ts
const server = Bun.serve({
  fetch(req, server) {
    return new Response(
      `Active requests: ${server.pendingRequests}\n` +
        `Active WebSockets: ${server.pendingWebSockets}`,
    );
  },
});
```

### server.subscriberCount(topic) - WebSocket 订阅者

获取 WebSocket 主题的订阅者数量：

```ts
const server = Bun.serve({
  fetch(req, server) {
    const chatUsers = server.subscriberCount("chat");
    return new Response(`${chatUsers} users in chat`);
  },
  websocket: {
    message(ws) {
      ws.subscribe("chat");
    },
  },
});
```

## WebSocket 配置

### server.publish(topic, data, compress) - WebSocket 消息发布

服务器可以向订阅主题的所有 WebSocket 客户端发布消息：

```ts
const server = Bun.serve({
  websocket: {
    message(ws) {
      // 发布到所有 "chat" 订阅者
      server.publish("chat", "Hello everyone!");
    },
  },

  fetch(req) {
    // ...
  },
});
```

`publish()` 方法返回：

- 如果成功，返回发送的字节数
- 如果消息被丢弃，返回 `0`
- 如果应用了背压，返回 `-1`

### WebSocket 处理器选项

配置 WebSocket 时，通过 `websocket` 处理器可以使用几个高级选项：

```ts
Bun.serve({
  websocket: {
    // 最大消息大小（字节）
    maxPayloadLength: 64 * 1024,

    // 在消息被丢弃之前的背压限制
    backpressureLimit: 1024 * 1024,

    // 当达到背压限制时关闭连接
    closeOnBackpressureLimit: true,

    // 当背压缓解时调用的处理器
    drain(ws) {
      console.log("Backpressure relieved");
    },

    // 启用每消息 deflate 压缩
    perMessageDeflate: {
      compress: true,
      decompress: true,
    },

    // 发送 ping 帧以保持连接存活
    sendPings: true,

    // ping/pong 帧的处理器
    ping(ws, data) {
      console.log("Received ping");
    },
    pong(ws, data) {
      console.log("Received pong");
    },

    // 服务器是否接收自己发布的消息
    publishToSelf: false,
  },
});
```

## 基准测试

以下是 Bun 和 Node.js 实现的简单 HTTP 服务器，对每个传入的 `Request` 响应 `Bun!`。

{% codetabs %}

```ts#Bun
Bun.serve({
  fetch(req: Request) {
    return new Response("Bun!");
  },
  port: 3000,
});
```

```ts#Node
require("http")
  .createServer((req, res) => res.end("Bun!"))
  .listen(8080);
```

{% /codetabs %}

在 Linux 上，`Bun.serve` 服务器每秒可以处理的请求数大约是 Node.js 的 2.5 倍。

{% table %}

- 运行时
- 每秒请求数

---

- Node 16
- ~64,000

---

- Bun
- ~160,000

{% /table %}

{% image width="499" alt="image" src="https://user-images.githubusercontent.com/709451/162389032-fc302444-9d03-46be-ba87-c12bd8ce89a0.png" /%}

## 参考

{% details summary="查看 TypeScript 定义" %}

```ts
interface Server extends Disposable {
  /**
   * 停止服务器接受新连接。
   * @param closeActiveConnections 如果为 true，立即终止所有连接
   * @returns 当服务器停止时解析的 Promise
   */
  stop(closeActiveConnections?: boolean): Promise<void>;

  /**
   * 更新处理器而无需重启服务器。
   * 只能更新 fetch 和 error 处理器。
   */
  reload(options: Serve): void;

  /**
   * 向运行中的服务器发出请求。
   * 对测试或内部路由很有用。
   */
  fetch(request: Request | string): Response | Promise<Response>;

  /**
   * 将 HTTP 请求升级为 WebSocket 连接。
   * @returns 如果升级成功则为 true，如果失败则为 false
   */
  upgrade<T = undefined>(
    request: Request,
    options?: {
      headers?: Bun.HeadersInit;
      data?: T;
    },
  ): boolean;

  /**
   * 向订阅主题的所有 WebSocket 客户端发布消息。
   * @returns 发送的字节数，如果丢弃则为 0，如果应用了背压则为 -1
   */
  publish(
    topic: string,
    data: string | ArrayBufferView | ArrayBuffer | SharedArrayBuffer,
    compress?: boolean,
  ): ServerWebSocketSendStatus;

  /**
   * 获取订阅主题的 WebSocket 客户端数量。
   */
  subscriberCount(topic: string): number;

  /**
   * 获取客户端 IP 地址和端口。
   * @returns 对于已关闭的请求或 Unix 套接字返回 null
   */
  requestIP(request: Request): SocketAddress | null;

  /**
   * 为请求设置自定义空闲超时。
   * @param seconds 超时秒数，0 表示禁用
   */
  timeout(request: Request, seconds: number): void;

  /**
   * 在服务器运行时保持进程存活。
   */
  ref(): void;

  /**
   * 如果服务器是唯一运行的东西，允许进程退出。
   */
  unref(): void;

  /** 正在进行的 HTTP 请求数 */
  readonly pendingRequests: number;

  /** 活动的 WebSocket 连接数 */
  readonly pendingWebSockets: number;

  /** 服务器 URL，包括协议、主机名和端口 */
  readonly url: URL;

  /** 服务器监听的端口 */
  readonly port: number;

  /** 服务器绑定的主机名 */
  readonly hostname: string;

  /** 服务器是否处于开发模式 */
  readonly development: boolean;

  /** 服务器实例标识符 */
  readonly id: string;
}

interface WebSocketHandler<T = undefined> {
  /** 最大 WebSocket 消息大小（字节） */
  maxPayloadLength?: number;

  /** 应用背压之前的排队消息字节数 */
  backpressureLimit?: number;

  /** 是否在达到背压限制时关闭连接 */
  closeOnBackpressureLimit?: boolean;

  /** 当背压缓解时调用 */
  drain?(ws: ServerWebSocket<T>): void | Promise<void>;

  /** 空闲超时秒数 */
  idleTimeout?: number;

  /** 启用每消息 deflate 压缩 */
  perMessageDeflate?:
    | boolean
    | {
        compress?: WebSocketCompressor | boolean;
        decompress?: WebSocketCompressor | boolean;
      };

  /** 发送 ping 帧以保持连接存活 */
  sendPings?: boolean;

  /** 服务器是否接收自己发布的消息 */
  publishToSelf?: boolean;

  /** 连接打开时调用 */
  open?(ws: ServerWebSocket<T>): void | Promise<void>;

  /** 收到消息时调用 */
  message(
    ws: ServerWebSocket<T>,
    message: string | Buffer,
  ): void | Promise<void>;

  /** 连接关闭时调用 */
  close?(
    ws: ServerWebSocket<T>,
    code: number,
    reason: string,
  ): void | Promise<void>;

  /** 收到 ping 帧时调用 */
  ping?(ws: ServerWebSocket<T>, data: Buffer): void | Promise<void>;

  /** 收到 pong 帧时调用 */
  pong?(ws: ServerWebSocket<T>, data: Buffer): void | Promise<void>;
}

interface TLSOptions {
  /** 证书颁发机构链 */
  ca?: string | Buffer | BunFile | Array<string | Buffer | BunFile>;

  /** 服务器证书 */
  cert?: string | Buffer | BunFile | Array<string | Buffer | BunFile>;

  /** DH 参数文件路径 */
  dhParamsFile?: string;

  /** 私钥 */
  key?: string | Buffer | BunFile | Array<string | Buffer | BunFile>;

  /** 减少 TLS 内存使用 */
  lowMemoryMode?: boolean;

  /** 私钥密码短语 */
  passphrase?: string;

  /** OpenSSL 选项标志 */
  secureOptions?: number;

  /** SNI 的服务器名称 */
  serverName?: string;
}
```

{% /details %}
