`bun` CLI 可用于执行 JavaScript/TypeScript 文件、`package.json` 脚本和[可执行包](https://docs.npmjs.com/cli/v9/configuring-npm/package-json#bin)。

## 性能

Bun 的设计目标是快速启动和快速运行。

在底层，Bun 使用由苹果为 Safari 开发的 [JavaScriptCore 引擎](https://developer.apple.com/documentation/javascriptcore)。在大多数情况下，其启动和运行性能比 Node.js 和基于 Chromium 的浏览器使用的 V8 引擎更快。其转译器和运行时是用 Zig 编写的，这是一种现代的高性能语言。在 Linux 上，这意味着启动时间比 Node.js [快 4 倍](https://twitter.com/jarredsumner/status/1499225725492076544)。

{% table %}

---

- `bun hello.js`
- `5.2ms`

---

- `node hello.js`
- `25.1ms`

{% /table %}
{% caption content="在 Linux 上运行简单的 Hello World 脚本" /%}

<!-- {% image src="/images/bun-run-speed.jpeg" caption="Bun vs Node.js vs Deno running Hello World" /%} -->

<!-- ## Speed -->

<!--
Performance sensitive APIs like `Buffer`, `fetch`, and `Response` are heavily profiled and optimized. Under the hood Bun uses the [JavaScriptCore engine](https://developer.apple.com/documentation/javascriptcore), which is developed by Apple for Safari. It starts and runs faster than V8, the engine used by Node.js and Chromium-based browsers. -->

## 运行文件

{% callout %}
对比 `node <file>`
{% /callout %}

使用 `bun run` 来执行源文件。

```bash
$ bun run index.js
```

Bun 默认支持 TypeScript 和 JSX。每个文件在执行前都会通过 Bun 的快速原生转译器进行即时转译。

```bash
$ bun run index.js
$ bun run index.jsx
$ bun run index.ts
$ bun run index.tsx
```

另外，您可以省略 `run` 关键字，直接使用"裸"命令；它的行为完全相同。

```bash
$ bun index.tsx
$ bun index.js
```

### `--watch`

要以监视模式运行文件，请使用 `--watch` 标志。

```bash
$ bun --watch run index.tsx
```

{% callout %}
**注意** — 使用 `bun run` 时，将 Bun 的标志（如 `--watch`）直接放在 `bun` 后面。

```bash
$ bun --watch run dev # ✔️ 这样做
$ bun run dev --watch # ❌ 不要这样做
```

出现在命令末尾的标志将被忽略，并传递给 `"dev"` 脚本本身。
{% /callout %}

## 运行 `package.json` 脚本

{% note %}
对比 `npm run <script>` 或 `yarn <script>`
{% /note %}

```sh
$ bun [bun flags] run <script> [script flags]
```

您的 `package.json` 可以定义多个命名的 `"scripts"`，它们对应于 shell 命令。

```json
{
  // ... other fields
  "scripts": {
    "clean": "rm -rf dist && echo 'Done.'",
    "dev": "bun server.ts"
  }
}
```

使用 `bun run <script>` 来执行这些脚本。

```bash
$ bun run clean
 $ rm -rf dist && echo 'Done.'
 Cleaning...
 Done.
```

Bun 在子 shell 中执行脚本命令。在 Linux 和 macOS 上，它会按顺序检查以下 shell，使用找到的第一个：`bash`、`sh`、`zsh`。在 Windows 上，它使用 [bun shell](https://bun.sh/docs/runtime/shell) 来支持类似 bash 的语法和许多常用命令。

{% callout %}
⚡️ 在 Linux 上，`npm run` 的启动时间约为 170ms；而使用 Bun 时仅为 `6ms`。
{% /callout %}

脚本也可以使用更短的命令 `bun <script>` 运行，但是如果有同名的内置 bun 命令，则内置命令优先。在这种情况下，使用更明确的 `bun run <script>` 命令来执行您的包脚本。

```bash
$ bun run dev
```

要查看可用脚本列表，请不带参数运行 `bun run`。

```bash
$ bun run
quickstart scripts:

 bun run clean
   rm -rf dist && echo 'Done.'

 bun run dev
   bun server.ts

2 scripts
```

Bun 尊重生命周期钩子。例如，`bun run clean` 将执行 `preclean` 和 `postclean`（如果已定义）。如果 `pre<script>` 失败，Bun 将不会执行脚本本身。

### `--bun`

`package.json` 脚本通常引用本地安装的 CLI，如 `vite` 或 `next`。这些 CLI 通常是带有 [shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) 的 JavaScript 文件，表明它们应该用 `node` 执行。

```js
#!/usr/bin/env node

// do stuff
```

默认情况下，Bun 尊重这个 shebang 并使用 `node` 执行脚本。但是，您可以使用 `--bun` 标志覆盖此行为。对于基于 Node.js 的 CLI，这将使用 Bun 而不是 Node.js 运行 CLI。

```bash
$ bun run --bun vite
```

### 过滤

在包含多个包的 monorepo 中，您可以使用 `--filter` 参数同时执行多个包中的脚本。

使用 `bun run --filter <name_pattern> <script>` 在所有名称匹配 `<name_pattern>` 的包中执行 `<script>`。
例如，如果您有包含名为 `foo`、`bar` 和 `baz` 的包的子目录，运行

```bash
bun run --filter 'ba*' <script>
```

将在 `bar` 和 `baz` 中执行 `<script>`，但不会在 `foo` 中执行。

在 [filter](https://bun.sh/docs/cli/filter#running-scripts-with-filter) 文档页面中查找更多详细信息。

## `bun run -` 从标准输入读取代码

`bun run -` 允许您从标准输入读取 JavaScript、TypeScript、TSX 或 JSX，并在不先写入临时文件的情况下执行它。

```bash
$ echo "console.log('Hello')" | bun run -
Hello
```

您还可以使用 `bun run -` 将文件重定向到 Bun。例如，将 `.js` 文件作为 `.ts` 文件运行：

```bash
$ echo "console.log!('This is TypeScript!' as any)" > secretly-typescript.js
$ bun run - < secretly-typescript.js
This is TypeScript!
```

为了方便起见，使用 `bun run -` 时，所有代码都被视为支持 JSX 的 TypeScript。

## `bun run --smol`

在内存受限的环境中，使用 `--smol` 标志以减少内存使用，但会降低性能。

```bash
$ bun --smol run index.tsx
```

这会导致垃圾收集器更频繁地运行，这可能会减慢执行速度。但是，它在内存有限的环境中可能很有用。Bun 会根据可用内存（考虑 cgroups 和其他内存限制）自动调整垃圾收集器的堆大小，无论是否使用 `--smol` 标志，所以这主要用于希望堆大小增长更慢的情况。

## 解析顺序

绝对路径和以 `./` 或 `.\\` 开头的路径始终作为源文件执行。除非使用 `bun run`，否则运行具有允许扩展名的文件将优先选择文件而不是 package.json 脚本。

当存在 package.json 脚本和同名文件时，`bun run` 优先考虑 package.json 脚本。完整的解析顺序是：

1. package.json 脚本，例如 `bun run build`
2. 源文件，例如 `bun run src/main.js`
3. 项目包中的二进制文件，例如 `bun add eslint && bun run eslint`
4. （仅 `bun run`）系统命令，例如 `bun run ls`

{% bunCLIUsage command="run" /%}
