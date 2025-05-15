{% callout %}
**注意** — 使用 Bun 不需要 `bun create`。你根本不需要任何配置。这个命令的存在只是为了让你能更快更容易地开始。
{% /callout %}

使用 `bun create` 来模板化一个新的 Bun 项目。这是一个灵活的命令，可以用来从 React 组件、`create-<template>` npm 包、GitHub 仓库或本地模板创建新项目。

如果你想创建一个全新的空项目，请使用 [`bun init`](https://bun.sh/docs/cli/init)。

## 从 React 组件创建

`bun create ./MyComponent.tsx` 将现有的 React 组件转换为一个完整的开发环境，包含热重载和生产构建，只需一个命令。

```bash
$ bun create ./MyComponent.jsx # 也支持 .tsx
```

{% raw %}

<video style="aspect-ratio: 2062 / 1344; width: 100%; height: 100%; object-fit: contain;"  loop autoplay muted playsinline>
  <source src="/bun-create-shadcn.mp4" style="width: 100%; height: 100%; object-fit: contain;" type="video/mp4">
</video>

{% /raw %}

{% callout %}
🚀 **Create React App 的继任者** — `bun create <component>` 提供了开发者喜爱的 Create React App 的所有功能，但使用了现代工具、更快的构建速度和后端支持。
{% /callout %}

#### 工作原理

当你运行 `bun create <component>` 时，Bun 会：

1. 使用 [Bun 的 JavaScript 打包器](https://bun.sh/docs/bundler) 分析你的模块图。
2. 收集运行组件所需的所有依赖。
3. 扫描入口点的导出以查找 React 组件。
4. 生成一个包含运行组件所需依赖和脚本的 `package.json` 文件。
5. 使用 [`bun install --only-missing`](https://bun.sh/docs/cli/install) 安装任何缺失的依赖。
6. 生成以下文件：
   - `${component}.html`
   - `${component}.client.tsx`（前端的入口点）
   - `${component}.css`（css 文件）
7. 自动启动前端开发服务器。

### 在 Bun 中使用 TailwindCSS

[TailwindCSS](https://tailwindcss.com/) 是一个非常流行的实用优先的 CSS 框架，用于样式化 Web 应用程序。

当你运行 `bun create <component>` 时，Bun 会扫描你的 JSX/TSX 文件中的 TailwindCSS 类名（以及它导入的任何文件）。如果检测到 TailwindCSS 类名，它会在你的 `package.json` 中添加以下依赖：

```json#package.json
{
  "dependencies": {
    "tailwindcss": "^4",
    "bun-plugin-tailwind": "latest"
  }
}
```

我们还配置 `bunfig.toml` 以使用 Bun 的 TailwindCSS 插件与 `Bun.serve()`

```toml#bunfig.toml
[serve.static]
plugins = ["bun-plugin-tailwind"]
```

并在 `${component}.css` 文件顶部添加 `@import "tailwindcss";`：

```css#MyComponent.css
@import "tailwindcss";
```

### 在 Bun 中使用 `shadcn/ui`

[`shadcn/ui`](https://ui.shadcn.com/) 是一个非常流行的用于构建 Web 应用程序的组件库工具。

`bun create <component>` 扫描从 `@/components/ui` 导入的任何 shadcn/ui 组件。

如果找到任何组件，它会运行：

```bash
# 假设 bun 检测到从 @/components/ui/accordion 和 @/components/ui/button 的导入
$ bunx shadcn@canary add accordion button # 以及任何其他组件
```

由于 `shadcn/ui` 本身使用 TailwindCSS，`bun create` 还会将必要的 TailwindCSS 依赖添加到你的 `package.json` 中，并配置 `bunfig.toml` 以使用 Bun 的 TailwindCSS 插件与 `Bun.serve()`，如上所述。

此外，我们还设置以下内容：

- `tsconfig.json` 将 `"@/*"` 别名设置为 `"src/*"` 或 `.`（取决于是否存在 `src/` 目录）
- `components.json` 以便 shadcn/ui 知道这是一个 shadcn/ui 项目
- `styles/globals.css` 文件，以 shadcn/ui 期望的方式配置 Tailwind v4
- `${component}.build.ts` 文件，使用配置了 `bun-plugin-tailwind` 的组件进行生产构建

`bun create ./MyComponent.jsx` 是在本地运行由 [Claude](https://claude.ai) 或 ChatGPT 等 LLM 生成的代码的最简单方法之一。

## 从 `npm` 创建

```sh
$ bun create <template> [<destination>]
```

假设你没有同名的[本地模板](#from-a-local-template)，此命令将从 npm 下载并执行 `create-<template>` 包。以下两个命令的行为将完全相同：

```sh
$ bun create remix
$ bunx create-remix
```

有关完整文档和使用说明，请参阅相关 `create-<template>` 包的文档。

## 从 GitHub 创建

这将下载 GitHub 仓库的内容到磁盘。

```bash
$ bun create <user>/<repo>
$ bun create github.com/<user>/<repo>
```

可以选择为目标文件夹指定名称。如果未指定目标，将使用仓库名称。

```bash
$ bun create <user>/<repo> mydir
$ bun create github.com/<user>/<repo> mydir
```

Bun 将执行以下步骤：

- 下载模板
- 将所有模板文件复制到目标文件夹
- 使用 `bun install` 安装依赖。
- 初始化一个新的 Git 仓库。使用 `--no-git` 标志选择退出。
- 如果定义了模板的 `start` 脚本，则运行它。

{% callout %}
默认情况下，Bun 将_不会覆盖_任何现有文件。使用 `--force` 标志覆盖现有文件。
{% /callout %}

## 从本地模板创建

{% callout %}
**⚠️ 警告** — 与远程模板不同，使用本地模板运行 `bun create` 如果目标文件夹已存在，将删除整个目标文件夹！请小心。
{% /callout %}

Bun 的模板器可以扩展以支持在本地文件系统上定义的自定义模板。这些模板应该位于以下目录之一：

- `$HOME/.bun-create/<name>`：全局模板
- `<project root>/.bun-create/<name>`：项目特定模板

{% callout %}
**注意** — 你可以通过设置 `BUN_CREATE_DIR` 环境变量来自定义全局模板路径。
{% /callout %}

要创建本地模板，导航到 `$HOME/.bun-create` 并使用你想要的模板名称创建一个新目录。

```bash
$ cd $HOME/.bun-create
$ mkdir foo
$ cd foo
```

然后，在该目录中创建一个 `package.json` 文件，内容如下：

```json
{
  "name": "foo"
}
```

你可以在文件系统的其他地方运行 `bun create foo` 来验证 Bun 是否正确找到了你的本地模板。

#### 设置逻辑

你可以在本地模板的 `package.json` 的 `"bun-create"` 部分指定安装前和安装后的设置脚本。

```json
{
  "name": "@bun-examples/simplereact",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "react": "^17.0.2",
    "react-dom": "^17.0.2"
  },
  "bun-create": {
    "preinstall": "echo 'Installing...'", // 单个命令
    "postinstall": ["echo 'Done!'"], // 命令数组
    "start": "bun run echo 'Hello world!'"
  }
}
```

支持以下字段。每个字段可以对应一个字符串或字符串数组。命令数组将按顺序执行。

{% table %}

---

- `postinstall`
- 在安装依赖后运行

---

- `preinstall`
- 在安装依赖前运行

{% /table %}

克隆模板后，`bun create` 会在将更新后的 `package.json` 写入目标文件夹之前自动删除 `"bun-create"` 部分。

## 参考

### CLI 标志

{% table %}

- 标志
- 描述

---

- `--force`
- 覆盖现有文件

---

- `--no-install`
- 跳过安装 `node_modules` 和任务

---

- `--no-git`
- 不初始化 git 仓库

---

- `--open`
- 完成后启动并在浏览器中打开

{% /table %}

### 环境变量

{% table %}

- 名称
- 描述

---

- `GITHUB_API_DOMAIN`
- 如果你使用 GitHub 企业版或代理，你可以自定义 Bun 用于下载的 GitHub 域名

---

- `GITHUB_TOKEN`（或 `GITHUB_ACCESS_TOKEN`）
- 这允许 `bun create` 与私有仓库一起工作，或者在你被限制速率时使用。如果两者都存在，则选择 `GITHUB_TOKEN` 而不是 `GITHUB_ACCESS_TOKEN`。

{% /table %}

{% details summary="`bun create` 的工作原理" %}

当你运行 `bun create ${template} ${destination}` 时，会发生以下情况：

如果是远程模板

1. GET `registry.npmjs.org/@bun-examples/${template}/latest` 并解析它
2. GET `registry.npmjs.org/@bun-examples/${template}/-/${template}-${latestVersion}.tgz`
3. 解压并将 `${template}-${latestVersion}.tgz` 提取到 `${destination}`

   - 如果有要覆盖的文件，除非传递了 `--force`，否则会警告并退出

如果是 GitHub 仓库

1. 从 GitHub 的 API 下载 tarball
2. 解压并提取到 `${destination}`

   - 如果有要覆盖的文件，除非传递了 `--force`，否则会警告并退出

如果是本地模板

1. 打开本地模板文件夹
2. 递归删除目标目录
3. 使用最快的系统调用递归复制文件（在 macOS 上使用 `fcopyfile`，在 Linux 上使用 `copy_file_range`）。如果存在 `node_modules` 文件夹，则不复制或遍历（仅这一点就比 `cp` 快）

4. 解析 `package.json`（再次！），将 `name` 更新为 `${basename(destination)}`，从 `package.json` 中删除 `bun-create` 部分，并将更新后的 `package.json` 保存到磁盘。
   - 如果检测到 Next.js，将 `bun-framework-next` 添加到依赖列表中
   - 如果检测到 Create React App，将 /src/index.{js,jsx,ts,tsx} 中的入口点添加到 `public/index.html`
   - 如果检测到 Relay，添加 `bun-macro-relay` 以便 Relay 工作
5. 自动检测 npm 客户端，优先使用 `pnpm`、`yarn`（v1），最后是 `npm`
6. 使用 npm 客户端运行 `"bun-create": { "preinstall" }` 中定义的任何任务
7. 除非传递了 `--no-install` 或 package.json 中没有依赖，否则运行 `${npmClient} install`
8. 使用 npm 客户端运行 `"bun-create": { "postinstall" }` 中定义的任何任务
9. 运行 `git init; git add -A .; git commit -am "Initial Commit";`

   - 将 `gitignore` 重命名为 `.gitignore`。NPM 自动从包中删除 `.gitignore` 文件。
   - 如果有依赖，这会在安装 node_modules 的同时在单独的线程中并发运行
   - 测试了使用 libgit2，但在微基准测试中性能慢了 3 倍

{% /details %}
