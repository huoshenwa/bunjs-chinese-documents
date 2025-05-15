如果在工作目录或更高层级中找不到 `node_modules` 目录，Bun 将放弃 Node.js 风格的模块解析，转而使用 **Bun 模块解析算法**。

在 Bun 风格的模块解析下，所有导入的包都会在执行过程中自动安装到[全局模块缓存](https://bun.sh/docs/install/cache)中（与 [`bun install`](https://bun.sh/docs/cli/install) 使用的缓存相同）。

```ts
import { foo } from "foo"; // 安装 `latest` 版本

foo();
```

第一次运行此脚本时，Bun 将自动安装 `"foo"` 并缓存它。下次运行脚本时，它将使用缓存的版本。

## 版本解析

为了确定要安装哪个版本，Bun 遵循以下算法：

1. 检查项目根目录中是否存在 `bun.lock` 文件。如果存在，使用锁文件中指定的版本。
2. 否则，向上扫描树查找包含 `"foo"` 作为依赖项的 `package.json`。如果找到，使用指定的 semver 版本或版本范围。
3. 否则，使用 `latest`。

## 缓存行为

一旦确定了版本或版本范围，Bun 将：

1. 检查模块缓存中是否存在兼容版本。如果存在，使用它。
2. 在解析 `latest` 时，Bun 将检查 `package@latest` 是否在过去 _24 小时_ 内已下载并缓存。如果是，使用它。
3. 否则，从 `npm` 注册表下载并安装适当的版本。

## 安装

包被安装并缓存到 `<cache>/<pkg>@<version>`，因此同一包的多个版本可以同时缓存。此外，在 `<cache>/<pkg>/<version>` 下创建一个符号链接，以便更快地查找缓存中存在的包的所有版本。

## 版本说明符

通过直接在导入语句中指定版本或版本范围，可以绕过整个解析算法。

```ts
import { z } from "zod@3.0.0"; // 特定版本
import { z } from "zod@next"; // npm 标签
import { z } from "zod@^3.20.0"; // semver 范围
```

## 优势

这种自动安装方法有几个好处：

- **空间效率** — 每个依赖版本只存在于磁盘上的一个位置。与冗余的每项目安装相比，这大大节省了空间和时间。
- **可移植性** — 要共享简单的脚本和代码片段，您的源文件是_自包含的_。不需要将包含代码和配置文件的目录一起 `zip`。使用导入语句中的版本说明符，甚至不需要 `package.json`。
- **便利性** — 在运行文件或脚本之前不需要运行 `npm install` 或 `bun install`。只需 `bun run` 它。
- **向后兼容性** — 因为如果存在 `package.json`，Bun 仍然尊重其中指定的版本，您可以通过单个命令切换到 Bun 风格的解析：`rm -rf node_modules`。

## 限制

- 没有智能提示。IDE 中的 TypeScript 自动完成依赖于 `node_modules` 中类型声明文件的存在。我们正在研究各种解决方案。
- 不支持 [patch-package](https://github.com/ds300/patch-package)

## 常见问题

{% details summary="这与 pnpm 有什么不同？" %}

使用 pnpm，您必须运行 `pnpm install`，这会为运行时解析创建一个符号链接的 `node_modules` 文件夹。相比之下，Bun 在运行文件时动态解析依赖项；不需要提前运行任何 `install` 命令。Bun 也不会创建 `node_modules` 文件夹。

{% /details %}

{% details summary="这与 Yarn Plug'N'Play 有什么不同？" %}
使用 Yarn，您必须在运行脚本之前运行 `yarn install`。相比之下，Bun 在运行文件时动态解析依赖项；不需要提前运行任何 `install` 命令。

Yarn Plug'N'Play 还使用 zip 文件来存储依赖项。这使得依赖项加载[在运行时更慢](https://twitter.com/jarredsumner/status/1458207919636287490)，因为 zip 文件上的随机访问读取往往比等效的磁盘查找慢。
{% /details %}

{% details summary="这与 Deno 有什么不同？" %}

Deno 要求在每个 npm `import` 之前使用 `npm:` 说明符，缺乏通过 `tsconfig.json` 中的 `compilerOptions.paths` 支持导入映射，并且对 `package.json` 设置的支持不完整。与 Deno 不同，Bun 目前不支持 URL 导入。
{% /details %}
