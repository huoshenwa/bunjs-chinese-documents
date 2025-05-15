Bun 会自动读取您的 `.env` 文件，并提供以编程方式读取和写入环境变量的惯用方法。此外，Bun 运行时的某些行为可以通过 Bun 特定的环境变量进行配置。

## 设置环境变量

Bun 会自动读取以下文件（按优先级递增的顺序列出）。

- `.env`
- `.env.production`、`.env.development`、`.env.test`（取决于 `NODE_ENV` 的值）
- `.env.local`

```txt#.env
FOO=hello
BAR=world
```

也可以通过命令行设置变量。

{% codetabs %}

```sh#Linux/macOS
$ FOO=helloworld bun run dev
```

```sh#Windows
# 使用 CMD
$ set FOO=helloworld && bun run dev

# 使用 PowerShell
$ $env:FOO="helloworld"; bun run dev
```

{% /codetabs %}

{% details summary="Windows 的跨平台解决方案" %}

对于跨平台解决方案，您可以使用 [bun shell](https://bun.sh/docs/runtime/shell)。例如，`bun exec` 命令。

```sh
$ bun exec 'FOO=helloworld bun run dev'
```

在 Windows 上，使用 `bun run` 调用的 `package.json` 脚本将自动使用 **bun shell**，这使得以下内容也是跨平台的。

```json#package.json
"scripts": {
  "dev": "NODE_ENV=development bun --watch app.ts",
},
```

{% /details %}

或者通过为 `process.env` 分配属性以编程方式设置。

```ts
process.env.FOO = "hello";
```

### 手动指定 `.env` 文件

Bun 支持 `--env-file` 来覆盖要加载的特定 `.env` 文件。您可以在 bun 运行时运行脚本时使用 `--env-file`，或者在运行 package.json 脚本时使用。

```sh
$ bun --env-file=.env.1 src/index.ts

$ bun --env-file=.env.abc --env-file=.env.def run build
```

### 引号

Bun 支持双引号、单引号和模板字面量反引号：

```txt#.env
FOO='hello'
FOO="hello"
FOO=`hello`
```

### 展开

环境变量会自动_展开_。这意味着您可以在环境变量中引用先前定义的变量。

```txt#.env
FOO=world
BAR=hello$FOO
```

```ts
process.env.BAR; // => "helloworld"
```

这对于构建连接字符串或其他复合值很有用。

```txt#.env
DB_USER=postgres
DB_PASSWORD=secret
DB_HOST=localhost
DB_PORT=5432
DB_URL=postgres://$DB_USER:$DB_PASSWORD@$DB_HOST:$DB_PORT/$DB_NAME
```

可以通过用反斜杠转义 `$` 来禁用此功能。

```txt#.env
FOO=world
BAR=hello\$FOO
```

```ts
process.env.BAR; // => "hello$FOO"
```

### `dotenv`

一般来说，您不再需要 `dotenv` 或 `dotenv-expand`，因为 Bun 会自动读取 `.env` 文件。

## 读取环境变量

可以通过 `process.env` 访问当前的环境变量。

```ts
process.env.API_TOKEN; // => "secret"
```

Bun 还通过 `Bun.env` 和 `import.meta.env` 暴露这些变量，这是 `process.env` 的简单别名。

```ts
Bun.env.API_TOKEN; // => "secret"
import.meta.env.API_TOKEN; // => "secret"
```

要在命令行打印所有当前设置的环境变量，请运行 `bun --print process.env`。这对于调试很有用。

```sh
$ bun --print process.env
BAZ=stuff
FOOBAR=aaaaaa
<lots more lines>
```

## TypeScript

在 TypeScript 中，`process.env` 的所有属性都被类型化为 `string | undefined`。

```ts
Bun.env.whatever;
// string | undefined
```

要获得自动完成并告诉 TypeScript 将变量视为非可选字符串，我们将使用[接口合并](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#merging-interfaces)。

```ts
declare module "bun" {
  interface Env {
    AWESOME: string;
  }
}
```

将此行添加到项目中的任何文件。它将全局添加 `AWESOME` 属性到 `process.env` 和 `Bun.env`。

```ts
process.env.AWESOME; // => string
```

## 配置 Bun

这些环境变量由 Bun 读取并配置其行为的各个方面。

{% table %}

- 名称
- 描述

---

- `NODE_TLS_REJECT_UNAUTHORIZED`
- `NODE_TLS_REJECT_UNAUTHORIZED=0` 禁用 SSL 证书验证。这对于测试和调试很有用，但在生产环境中应该非常谨慎地使用此选项。注意：此环境变量最初由 Node.js 引入，我们保留名称以保持兼容性。

---

- `BUN_CONFIG_VERBOSE_FETCH`
- 如果 `BUN_CONFIG_VERBOSE_FETCH=curl`，则 fetch 请求将记录 url、方法、请求头和响应头到控制台。这对于调试网络请求很有用。这也适用于 `node:http`。`BUN_CONFIG_VERBOSE_FETCH=1` 等同于 `BUN_CONFIG_VERBOSE_FETCH=curl`，只是没有 `curl` 输出。

---

- `BUN_RUNTIME_TRANSPILER_CACHE_PATH`
- 运行时转译器缓存大于 50 kb 的源文件的转译输出。这使使用 Bun 的 CLI 加载更快。如果设置了 `BUN_RUNTIME_TRANSPILER_CACHE_PATH`，则运行时转译器将缓存转译输出到指定目录。如果 `BUN_RUNTIME_TRANSPILER_CACHE_PATH` 设置为空字符串或字符串 `"0"`，则运行时转译器将不会缓存转译输出。如果 `BUN_RUNTIME_TRANSPILER_CACHE_PATH` 未设置，则运行时转译器将缓存转译输出到平台特定的缓存目录。

---

- `TMPDIR`
- Bun 偶尔需要一个目录来存储打包或其他操作期间的中间资源。如果未设置，默认为平台特定的临时目录：Linux 上的 `/tmp`，macOS 上的 `/private/tmp`。

---

- `NO_COLOR`
- 如果 `NO_COLOR=1`，则禁用 [ANSI 颜色输出](https://no-color.org/)。

---

- `FORCE_COLOR`
- 如果 `FORCE_COLOR=1`，则强制启用 ANSI 颜色输出，即使设置了 `NO_COLOR`。

---

- `BUN_CONFIG_MAX_HTTP_REQUESTS`
- 控制 fetch 和 `bun install` 发送的最大并发 HTTP 请求数。默认为 `256`。如果您遇到速率限制或连接问题，可以减少此数字。

---

- `BUN_CONFIG_NO_CLEAR_TERMINAL_ON_RELOAD`
- 如果 `BUN_CONFIG_NO_CLEAR_TERMINAL_ON_RELOAD=true`，则 `bun --watch` 不会在重新加载时清除控制台

---

- `DO_NOT_TRACK`
- 在崩溃时禁用向 `bun.report` 上传崩溃报告。在 macOS 和 Windows 上，默认启用崩溃报告上传。否则，截至 2024 年 5 月 21 日，尚未发送遥测数据，但我们计划在未来几周内添加遥测功能。如果 `DO_NOT_TRACK=1`，则禁用自动上传崩溃报告和遥测数据。

{% /table %}

## 运行时转译器缓存

对于大于 50 KB 的文件，Bun 将转译输出缓存到 `$BUN_RUNTIME_TRANSPILER_CACHE_PATH` 或平台特定的缓存目录中。这使使用 Bun 的 CLI 加载更快。

此转译器缓存是全局的，并在所有项目之间共享。可以随时安全地删除缓存。它是一个基于内容的缓存，因此永远不会包含重复条目。在 Bun 进程运行时删除缓存也是安全的。

建议在使用 Docker 等临时文件系统时禁用此缓存。Bun 的 Docker 镜像会自动禁用此缓存。

### 禁用运行时转译器缓存

要禁用运行时转译器缓存，请将 `BUN_RUNTIME_TRANSPILER_CACHE_PATH` 设置为空字符串或字符串 `"0"`。

```sh
BUN_RUNTIME_TRANSPILER_CACHE_PATH=0 bun run dev
```

### 它缓存什么？

它缓存：

- 大于 50 KB 的源文件的转译输出。
- 文件转译输出的 sourcemap

这些缓存文件使用 `.pile` 文件扩展名。
