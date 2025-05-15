Bun 的打包器实现了 `--compile` 标志，用于从 TypeScript 或 JavaScript 文件生成独立的二进制文件。

{% codetabs %}

```bash
$ bun build ./cli.ts --compile --outfile mycli
```

```ts#cli.ts
console.log("Hello world!");
```

{% /codetabs %}

这将把 `cli.ts` 打包成一个可以直接执行的可执行文件：

```
$ ./mycli
Hello world!
```

所有导入的文件和包都被打包到可执行文件中，同时包含一份 Bun 运行时的副本。所有内置的 Bun 和 Node.js API 都受支持。

## 交叉编译到其他平台

`--target` 标志允许您为不同的操作系统、架构或 Bun 版本编译独立的可执行文件，而不是在运行 `bun build` 的机器上。

为 Linux x64（大多数服务器）构建：

```sh
bun build --compile --target=bun-linux-x64 ./index.ts --outfile myapp

# 要支持 2013 年之前的 CPU，使用基准版本（nehalem）
bun build --compile --target=bun-linux-x64-baseline ./index.ts --outfile myapp

# 要明确只支持 2013 年及以后的 CPU，使用现代版本（haswell）
# 现代版本更快，但基准版本兼容性更好。
bun build --compile --target=bun-linux-x64-modern ./index.ts --outfile myapp
```

为 Linux ARM64（例如 Graviton 或 Raspberry Pi）构建：

```sh
# 注意：如果未指定架构，默认架构是 x64。
bun build --compile --target=bun-linux-arm64 ./index.ts --outfile myapp
```

为 Windows x64 构建：

```sh
bun build --compile --target=bun-windows-x64 ./path/to/my/app.ts --outfile myapp

# 要支持 2013 年之前的 CPU，使用基准版本（nehalem）
bun build --compile --target=bun-windows-x64-baseline ./path/to/my/app.ts --outfile myapp

# 要明确只支持 2013 年及以后的 CPU，使用现代版本（haswell）
bun build --compile --target=bun-windows-x64-modern ./path/to/my/app.ts --outfile myapp

# 注意：如果没有提供 .exe 扩展名，Bun 会自动为 Windows 可执行文件添加它
```

为 macOS arm64 构建：

```sh
bun build --compile --target=bun-darwin-arm64 ./path/to/my/app.ts --outfile myapp
```

为 macOS x64 构建：

```sh
bun build --compile --target=bun-darwin-x64 ./path/to/my/app.ts --outfile myapp
```

#### 支持的目标

`--target` 标志的顺序并不重要，只要它们用 `-` 分隔即可。

| --target              | 操作系统 | 架构  | 现代版 | 基准版 | Libc  |
| --------------------- | -------- | ----- | ------ | ------ | ----- |
| bun-linux-x64         | Linux    | x64   | ✅     | ✅     | glibc |
| bun-linux-arm64       | Linux    | arm64 | ✅     | N/A    | glibc |
| bun-windows-x64       | Windows  | x64   | ✅     | ✅     | -     |
| ~~bun-windows-arm64~~ | Windows  | arm64 | ❌     | ❌     | -     |
| bun-darwin-x64        | macOS    | x64   | ✅     | ✅     | -     |
| bun-darwin-arm64      | macOS    | arm64 | ✅     | N/A    | -     |
| bun-linux-x64-musl    | Linux    | x64   | ✅     | ✅     | musl  |
| bun-linux-arm64-musl  | Linux    | arm64 | ✅     | N/A    | musl  |

在 x64 平台上，Bun 使用 SIMD 优化，这需要支持 AVX2 指令的现代 CPU。Bun 的 `-baseline` 构建适用于不支持这些优化的旧 CPU。通常，当您安装 Bun 时，我们会自动检测使用哪个版本，但在交叉编译时这可能更难做到，因为您可能不知道目标 CPU。在 Darwin x64 上通常不需要担心这个问题，但在 Windows x64 和 Linux x64 上这很重要。如果您或您的用户看到 `"Illegal instruction"` 错误，您可能需要使用基准版本。

## 部署到生产环境

编译的可执行文件可以减少内存使用并提高 Bun 的启动时间。

通常，Bun 在 `import` 和 `require` 时读取并转译 JavaScript 和 TypeScript 文件。这是使 Bun 如此"即插即用"的部分原因，但这并不是免费的。从磁盘读取文件、解析文件路径、解析、转译和打印源代码都需要时间和内存。

使用编译的可执行文件，您可以将这些成本从运行时转移到构建时。

在部署到生产环境时，我们建议以下做法：

```sh
bun build --compile --minify --sourcemap ./path/to/my/app.ts --outfile myapp
```

### 字节码编译

要改善启动时间，启用字节码编译：

```sh
bun build --compile --minify --sourcemap --bytecode ./path/to/my/app.ts --outfile myapp
```

使用字节码编译，`tsc` 启动速度提高 2 倍：

{% image src="https://github.com/user-attachments/assets/dc8913db-01d2-48f8-a8ef-ac4e984f9763" width="689" /%}

字节码编译将大型输入文件的解析开销从运行时转移到打包时。您的应用程序启动更快，代价是使 `bun build` 命令稍微慢一些。它不会混淆源代码。

**实验性：** 字节码编译是 Bun v1.1.30 中引入的实验性功能。仅支持 `cjs` 格式（这意味着不支持顶层 await）。如果您遇到任何问题，请告诉我们！

### 这些标志的作用是什么？

`--minify` 参数优化转译输出代码的大小。如果您有一个大型应用程序，这可以节省几兆字节的空间。对于较小的应用程序，它可能仍然会稍微改善启动时间。

`--sourcemap` 参数嵌入了一个用 zstd 压缩的源映射，这样错误和堆栈跟踪会指向它们原始的位置，而不是转译后的位置。当发生错误时，Bun 会自动解压缩和解析源映射。

`--bytecode` 参数启用字节码编译。每次在 Bun 中运行 JavaScript 代码时，JavaScriptCore（引擎）都会将您的源代码编译成字节码。我们可以将这种解析工作从运行时转移到打包时，为您节省启动时间。

## Worker

要在独立可执行文件中使用 workers，将 worker 的入口点添加到 CLI 参数中：

```sh
$ bun build --compile ./index.ts ./my-worker.ts --outfile myapp
```

然后，在您的代码中引用 worker：

```ts
console.log("Hello from Bun!");

// 以下任何一种方式都可以：
new Worker("./my-worker.ts");
new Worker(new URL("./my-worker.ts", import.meta.url));
new Worker(new URL("./my-worker.ts", import.meta.url).href);
```

从 Bun v1.1.25 开始，当您向独立可执行文件添加多个入口点时，它们将被分别打包到可执行文件中。

在未来，我们可能会自动检测 `new Worker(path)` 中静态已知路径的使用，然后将它们打包到可执行文件中，但目前，您需要像上面的示例那样手动将其添加到 shell 命令中。

如果您使用相对路径指向未包含在独立可执行文件中的文件，它将尝试从磁盘加载该路径，相对于进程的当前工作目录（如果不存在则会出错）。

## SQLite

您可以在 `bun build --compile` 中使用 `bun:sqlite` 导入。

默认情况下，数据库相对于进程的当前工作目录解析。

```js
import db from "./my.db" with { type: "sqlite" };

console.log(db.query("select * from users LIMIT 1").get());
```

这意味着如果可执行文件位于 `/usr/bin/hello`，用户的终端位于 `/home/me/Desktop`，它将查找 `/home/me/Desktop/my.db`。

```
$ cd /home/me/Desktop
$ ./hello
```

## 嵌入资源和文件

独立可执行文件支持嵌入文件。

要使用 `bun build --compile` 将文件嵌入到可执行文件中，在您的代码中导入文件：

```ts
// 这变成了一个内部文件路径
import icon from "./icon.png" with { type: "file" };
import { file } from "bun";

export default {
  fetch(req) {
    // 嵌入的文件可以从 Response 对象中流式传输
    return new Response(file(icon));
  },
};
```

嵌入的文件可以使用 `Bun.file` 的函数或 Node.js `fs.readFile` 函数（在 `"node:fs"` 中）读取。

例如，要读取嵌入文件的内容：

```js
import icon from "./icon.png" with { type: "file" };
import { file } from "bun";

const bytes = await file(icon).arrayBuffer();
// await fs.promises.readFile(icon)
// fs.readFileSync(icon)
```

### 嵌入 SQLite 数据库

如果您的应用程序想要嵌入 SQLite 数据库，在导入属性中设置 `type: "sqlite"` 并将 `embed` 属性设置为 `"true"`。

```js
import myEmbeddedDb from "./my.db" with { type: "sqlite", embed: "true" };

console.log(myEmbeddedDb.query("select * from users LIMIT 1").get());
```

这个数据库是可读写的，但当可执行文件退出时所有更改都会丢失（因为它存储在内存中）。

### 嵌入 N-API 插件

从 Bun v1.0.23 开始，您可以将 `.node` 文件嵌入到可执行文件中。

```js
const addon = require("./addon.node");

console.log(addon.hello());
```

不幸的是，如果您使用 `@mapbox/node-pre-gyp` 或其他类似工具，您需要确保 `.node` 文件被直接 require，否则它将无法正确打包。

### 嵌入目录

要使用 `bun build --compile` 嵌入目录，在您的 `bun build` 命令中使用 shell glob：

```sh
$ bun build --compile ./index.ts ./public/**/*.png
```

然后，您可以在代码中引用这些文件：

```ts
import icon from "./public/assets/icon.png" with { type: "file" };
import { file } from "bun";

export default {
  fetch(req) {
    // 嵌入的文件可以从 Response 对象中流式传输
    return new Response(file(icon));
  },
};
```

这实际上是一个变通方案，我们期望在未来通过更直接的 API 来改进这一点。

### 列出嵌入的文件

要获取所有嵌入文件的列表，使用 `Bun.embeddedFiles`：

```js
import "./icon.png" with { type: "file" };
import { embeddedFiles } from "bun";

console.log(embeddedFiles[0].name); // `icon-${hash}.png`
```

`Bun.embeddedFiles` 返回一个 `Blob` 对象数组，您可以使用它来获取文件的大小、内容和其他属性。

```ts
embeddedFiles: Blob[]
```

嵌入文件的列表不包括像 `.ts` 和 `.js` 文件这样的打包源代码。

#### 内容哈希

默认情况下，嵌入的文件名称后附加了内容哈希。这对于您想要从 URL 或 CDN 提供文件并减少缓存失效问题的情况很有用。但有时，这是意外的，您可能想要原始名称：

要禁用内容哈希，向 `bun build --compile` 传递 `--asset-naming`，如下所示：

```sh
$ bun build --compile --asset-naming="[name].[ext]" ./index.ts
```

## 压缩

要稍微减小可执行文件的大小，向 `bun build --compile` 传递 `--minify`。这使用 Bun 的压缩器来减小代码大小。总的来说，Bun 的二进制文件仍然太大，我们需要使其更小。

## Windows 特定标志

在 Windows 上编译独立可执行文件时，有两个平台特定的选项可用于自定义生成的 `.exe` 文件的元数据：

- `--windows-icon=path/to/icon.ico` 自定义可执行文件图标。
- `--windows-hide-console` 禁用后台终端，可用于不需要 TTY 的应用程序。

{% callout %}

这些标志目前在交叉编译时不能使用，因为它们依赖于 Windows API。

{% /callout %}

## macOS 上的代码签名

要在 macOS 上对独立可执行文件进行代码签名（修复 Gatekeeper 警告），使用 `codesign` 命令。

```sh
$ codesign --deep --force -vvvv --sign "XXXXXXXXXX" ./myapp
```

我们建议包含一个带有 JIT 权限的 `entitlements.plist` 文件。

```xml#entitlements.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    <key>com.apple.security.cs.disable-executable-page-protection</key>
    <true/>
    <key>com.apple.security.cs.allow-dyld-environment-variables</key>
    <true/>
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
</dict>
</plist>
```

要使用 JIT 支持进行代码签名，向 `codesign` 传递 `--entitlements` 标志。

```sh
$ codesign --deep --force -vvvv --sign "XXXXXXXXXX" --entitlements entitlements.plist ./myapp
```

代码签名后，验证可执行文件：

```sh
$ codesign -vvv --verify ./myapp
./myapp: valid on disk
./myapp: satisfies its Designated Requirement
```

{% callout %}

代码签名支持需要 Bun v1.2.4 或更新版本。

{% /callout %}

## 不支持的 CLI 参数

目前，`--compile` 标志一次只能接受一个入口点，并且不支持以下标志：

- `--outdir` — 使用 `outfile` 代替。
- `--splitting`
- `--public-path`
- `--target=node` 或 `--target=browser`
- `--format` - 始终输出二进制可执行文件。内部几乎都是 esm。
- `--no-bundle` - 我们总是将所有内容打包到可执行文件中。
