让我们使用内置的 `Bun.serve` API 编写一个简单的 HTTP 服务器。首先，创建一个新目录。

```bash
$ mkdir quickstart
$ cd quickstart
```

运行 `bun init` 来搭建一个新项目。这是一个交互式工具；对于本教程，只需按 `enter` 接受每个提示的默认答案。

```bash
$ bun init
bun init 帮助您开始一个最小项目，并尝试
猜测合理的默认值。随时按 ^C 退出。

package name (quickstart):
entry point (index.ts):

完成！package.json 文件已保存在当前目录中。
 + index.ts
 + .gitignore
 + tsconfig.json（用于编辑器自动完成）
 + README.md

要开始使用，请运行：
  bun run index.ts
```

由于我们的入口点是 `*.ts` 文件，Bun 会为您生成一个 `tsconfig.json`。如果您使用纯 JavaScript，它将生成一个 [`jsconfig.json`](https://code.visualstudio.com/docs/languages/jsconfig)。

## 运行文件

打开 `index.ts` 并粘贴以下代码片段，它使用 [`Bun.serve`](https://bun.sh/docs/api/http) 实现了一个简单的 HTTP 服务器。

```ts
const server = Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response("Bun!");
  },
});

console.log(`Listening on http://localhost:${server.port} ...`);
```

{% details summary="在 `Bun` 上看到 TypeScript 错误？" %}
如果您使用了 `bun init`，Bun 将自动安装 Bun 的 TypeScript 声明并配置您的 `tsconfig.json`。如果您在现有项目中尝试 Bun，您可能会在 `Bun` 全局变量上看到类型错误。

要修复这个问题，首先将 `@types/bun` 作为开发依赖安装。

```sh
$ bun add -d @types/bun
```

然后在 `tsconfig.json` 的 `compilerOptions` 中添加以下内容：

```json#tsconfig.json
{
  "compilerOptions": {
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "Preserve",
    "moduleDetection": "force",
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
  }
}
```

{% /details %}

从您的 shell 运行该文件。

```bash
$ bun index.ts
Listening on http://localhost:3000 ...
```

访问 [http://localhost:3000](http://localhost:3000) 测试服务器。您应该看到一个显示 "Bun!" 的简单页面。

## 运行脚本

Bun 还可以执行 `package.json` 中的 `"scripts"`。添加以下脚本：

```json-diff
  {
    "name": "quickstart",
    "module": "index.ts",
    "type": "module",
+   "scripts": {
+     "start": "bun run index.ts"
+   },
    "devDependencies": {
      "@types/bun": "latest"
    }
  }
```

然后使用 `bun run start` 运行它。

```bash
$ bun run start
  $ bun run index.ts
  Listening on http://localhost:3000 ...
```

{% callout %}
⚡️ **性能** — `bun run` 比 `npm run` 快约 28 倍（开销为 6ms vs 170ms）。
{% /callout %}

## 安装包

让我们通过安装一个包来使我们的服务器更有趣。首先安装 `figlet` 包及其类型声明。Figlet 是一个用于将字符串转换为 ASCII 艺术的实用工具。

```bash
$ bun add figlet
$ bun add -d @types/figlet # 仅 TypeScript 用户
```

更新 `index.ts` 以在 `fetch` 处理程序中使用 `figlet`。

```ts-diff
+ import figlet from "figlet";

  const server = Bun.serve({
    port: 3000,
    fetch(req) {
+     const body = figlet.textSync("Bun!");
+     return new Response(body);
-     return new Response("Bun!");
    },
  });
```

重启服务器并刷新页面。您应该看到一个新的 ASCII 艺术横幅。

```txt
  ____              _
 | __ ) _   _ _ __ | |
 |  _ \| | | | '_ \| |
 | |_) | |_| | | | |_|
 |____/ \__,_|_| |_(_)
```
