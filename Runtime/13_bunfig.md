可以使用 Bun 的配置文件 `bunfig.toml` 来配置 Bun 的行为。

通常，Bun 依赖于现有的配置文件（如 `package.json` 和 `tsconfig.json`）来配置其行为。`bunfig.toml` 仅用于配置 Bun 特定的内容。此文件是可选的，没有它 Bun 也可以正常工作。

## 全局与本地

通常，建议在项目根目录（与 `package.json` 一起）添加 `bunfig.toml` 文件。

要全局配置 Bun，您也可以在以下路径之一创建 `.bunfig.toml` 文件：

- `$HOME/.bunfig.toml`
- `$XDG_CONFIG_HOME/.bunfig.toml`

如果同时检测到全局和本地 `bunfig`，结果会进行浅合并，本地配置会覆盖全局配置。在适用的情况下，CLI 标志会覆盖 `bunfig` 设置。

## 运行时

Bun 的运行时行为使用 `bunfig.toml` 文件中的顶级字段进行配置。

### `preload`

在运行文件或脚本之前要执行的脚本/插件数组。

```toml
# 在 `bun run` 文件或脚本之前运行的脚本
# 通过将它们添加到此列表来注册插件
preload = ["./preload.ts"]
```

### `jsx`

配置 Bun 如何处理 JSX。您也可以在 `tsconfig.json` 的 `compilerOptions` 中设置这些字段，但对于非 TypeScript 项目，这里也支持它们。

```toml
jsx = "react"
jsxFactory = "h"
jsxFragment = "Fragment"
jsxImportSource = "react"
```

有关这些字段的更多信息，请参考 tsconfig 文档。

- [jsx](https://www.typescriptlang.org/tsconfig#jsx)
- [jsxFactory](https://www.typescriptlang.org/tsconfig#jsxFactory)
- [jsxFragment](https://www.typescriptlang.org/tsconfig#jsxFragment)
- [jsxImportSource](https://www.typescriptlang.org/tsconfig#jsxImportSource)

### `smol`

启用 `smol` 模式。这会以性能为代价减少内存使用。

```toml
# 以性能为代价减少内存使用
smol = true
```

### `logLevel`

设置日志级别。这可以是 `"debug"`、`"warn"` 或 `"error"` 之一。

```toml
logLevel = "debug" # "debug" | "warn" | "error"
```

### `define`

`define` 字段允许您用常量表达式替换某些全局标识符。Bun 将用表达式替换标识符的任何使用。表达式应该是 JSON 字符串。

```toml
[define]
# 用字符串 `lox` 替换 `process.env.bagel` 的任何使用。
# 值被解析为 JSON，但支持单引号字符串，`'undefined'` 在 JS 中变为 `undefined`。
# 这可能在未来的版本中改变为只是普通的 TOML。它是 CLI 参数解析的遗留物。
"process.env.bagel" = "'lox'"
```

### `loader`

配置 Bun 如何将文件扩展名映射到加载器。这对于加载 Bun 不原生支持的文件很有用。

```toml
[loader]
# 当导入 .bagel 文件时，将其视为 tsx 文件
".bagel" = "tsx"
```

Bun 支持以下加载器：

- `jsx`
- `js`
- `ts`
- `tsx`
- `css`
- `file`
- `json`
- `toml`
- `wasm`
- `napi`
- `base64`
- `dataurl`
- `text`

### `telemetry`

`telemetry` 字段允许启用/禁用分析记录。Bun 记录打包时间（这样我们可以用数据回答"Bun 是否变得更快了？"）和功能使用情况（例如，"人们是否真的在使用宏？"）。请求体大小约为 60 字节，所以数据量不大。默认情况下启用遥测。相当于 `DO_NOT_TRACK` 环境变量。

```toml
telemetry = false
```

## 测试运行器

测试运行器在 `bunfig.toml` 的 `[test]` 部分下配置。

```toml
[test]
# 配置放在这里
```

### `test.root`

运行测试的根目录。默认为 `.`。

```toml
[test]
root = "./__tests__"
```

### `test.preload`

与顶级 `preload` 字段相同，但仅适用于 `bun test`。

```toml
[test]
preload = ["./setup.ts"]
```

### `test.smol`

与顶级 `smol` 字段相同，但仅适用于 `bun test`。

```toml
[test]
smol = true
```

### `test.coverage`

启用覆盖率报告。默认为 `false`。使用 `--coverage` 覆盖。

```toml
[test]
coverage = false
```

### `test.coverageThreshold`

指定覆盖率阈值。默认情况下不设置阈值。如果您的测试套件未达到或超过此阈值，`bun test` 将以非零退出代码退出以指示失败。

```toml
[test]

# 要求 90% 的行级和函数级覆盖率
coverageThreshold = 0.9
```

可以为行级、函数级和语句级覆盖率指定不同的阈值。

```toml
[test]
coverageThreshold = { line = 0.7, function = 0.8, statement = 0.9 }
```

### `test.coverageSkipTestFiles`

计算覆盖率统计信息时是否跳过测试文件。默认为 `false`。

```toml
[test]
coverageSkipTestFiles = false
```

### `test.coverageReporter`

默认情况下，覆盖率报告将打印到控制台。对于 CI 环境中的持久代码覆盖率报告和其他工具，使用 `lcov`。

```toml
[test]
coverageReporter  = ["text", "lcov"]  # 默认 ["text"]
```

### `test.coverageDir`

设置保存覆盖率报告的路径。请注意，这只适用于持久性 `coverageReporter`，如 `lcov`。

```toml
[test]
coverageDir = "path/to/somewhere"  # 默认 "coverage"
```

## 包管理器

包管理是一个复杂的问题；为了支持各种用例，可以在 `[install]` 部分下配置 `bun install` 的行为。

```toml
[install]
# 配置放在这里
```

### `install.optional`

是否安装可选依赖项。默认为 `true`。

```toml
[install]
optional = true
```

### `install.dev`

是否安装开发依赖项。默认为 `true`。

```toml
[install]
dev = true
```

### `install.peer`

是否安装对等依赖项。默认为 `true`。

```toml
[install]
peer = true
```

### `install.production`

`bun install` 是否在"生产模式"下运行。默认为 `false`。

在生产模式下，不安装 `"devDependencies"`。您可以在 CLI 中使用 `--production` 覆盖此设置。

```toml
[install]
production = false
```

### `install.exact`

是否在 `package.json` 中设置精确版本。默认为 `false`。

默认情况下，Bun 使用脱字符范围；如果包的 `latest` 版本是 `2.4.1`，您的 `package.json` 中的版本范围将是 `^2.4.1`。这表示从 `2.4.1` 到（但不包括）`3.0.0` 的任何版本都是可接受的。

```toml
[install]
exact = false
```

### `install.saveTextLockfile`

如果为 false，在运行 `bun install` 且不存在锁文件时，生成二进制 `bun.lockb` 而不是基于文本的 `bun.lock` 文件。

自 Bun v1.2 起默认为 `true`。

```toml
[install]
saveTextLockfile = false
```

### `install.auto`

配置 Bun 的包自动安装行为。默认为 `"auto"` — 当找不到 `node_modules` 文件夹时，Bun 将在执行过程中自动安装依赖项。

```toml
[install]
auto = "auto"
```

有效值：

{% table %}

- 值
- 描述

---

- `"auto"`
- 如果存在，从本地 `node_modules` 解析模块。否则，动态自动安装依赖项。

---

- `"force"`
- 始终自动安装依赖项，即使存在 `node_modules`。

---

- `"disable"`
- 从不自动安装依赖项。

---

- `"fallback"`
- 首先检查本地 `node_modules`，然后自动安装任何未找到的包。您可以使用 `bun -i` 从 CLI 启用此功能。

{% /table %}

### `install.frozenLockfile`

当为 true 时，`bun install` 不会更新 `bun.lock`。默认为 `false`。如果 `package.json` 和现有的 `bun.lock` 不一致，这将出错。

```toml
[install]
frozenLockfile = false
```

### `install.dryRun`

`bun install` 是否实际安装依赖项。默认为 `false`。当为 true 时，相当于在所有 `bun install` 命令上设置 `--dry-run`。

```toml
[install]
dryRun = false
```

### `install.globalDir`

配置 Bun 放置全局安装包的目录。

```toml
[install]
# `bun install --global` 安装包的位置
globalDir = "~/.bun/install/global"
```

### `install.globalBinDir`

配置 Bun 安装全局安装的二进制文件和 CLI 的目录。

```toml
# 全局安装的包二进制文件链接的位置
globalBinDir = "~/.bun/bin"
```

### `install.registry`

默认注册表是 `https://registry.npmjs.org/`。这可以在 `bunfig.toml` 中全局配置：

```toml
[install]
# 将默认注册表设置为字符串
registry = "https://registry.npmjs.org"
# 设置令牌
registry = { url = "https://registry.npmjs.org", token = "123456" }
# 设置用户名/密码
registry = "https://username:password@registry.npmjs.org"
```

### `install.scopes`

要为特定作用域（例如 `@myorg/<package>`）配置注册表，使用 `install.scopes`。您可以使用 `$variable` 表示法引用环境变量。

```toml
[install.scopes]
# 注册表作为字符串
myorg = "https://username:password@registry.myorg.com/"

# 带用户名/密码的注册表
# 您可以引用环境变量
myorg = { username = "myusername", password = "$npm_password", url = "https://registry.myorg.com/" }

# 带令牌的注册表
myorg = { token = "$npm_token", url = "https://registry.myorg.com/" }
```

### `install.ca` 和 `install.cafile`

要配置 CA 证书，使用 `install.ca` 或 `install.cafile` 指定 CA 证书文件的路径。

```toml
[install]
# CA 证书作为字符串
ca = "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----"

# CA 证书文件的路径。该文件可以包含多个证书。
cafile = "path/to/cafile"
```

### `install.cache`

配置缓存行为：

```toml
[install.cache]

# 用于缓存的目录
dir = "~/.bun/install/cache"

# 当为 true 时，不从全局缓存加载。
# Bun 可能仍会写入 node_modules/.cache
disable = false

# 当为 true 时，始终从注册表解析最新版本
disableManifest = false
```

### `install.lockfile`

要配置锁文件行为，使用 `install.lockfile` 部分。

是否在 `bun install` 上生成锁文件。默认为 `true`。

```toml
[install.lockfile]
save = true
```

是否在 `bun.lock` 旁边生成非 Bun 锁文件。（始终会创建 `bun.lock`。）目前 `"yarn"` 是唯一支持的值。

```toml
[install.lockfile]
print = "yarn"
```

## `bun run`

`bun run` 命令可以在 `[run]` 部分下配置。这些适用于 `bun run` 命令和运行文件或可执行文件或脚本时的 `bun` 命令。

目前，`bunfig.toml` 并不总是自动为本地项目中的 `bun run` 加载（它确实检查全局 `bunfig.toml`），所以您可能仍然需要传递 `-c` 或 `-c=bunfig.toml` 来使用这些设置。

### `run.shell` - 使用系统 shell 或 Bun 的 shell

运行 `package.json` 脚本时使用的 shell。在 Windows 上，这默认为 `"bun"`，在其他平台上默认为 `"system"`。

要始终使用系统 shell 而不是 Bun 的 shell（Windows 以外的默认行为）：

```toml
[run]
# Windows 以外的默认值
shell = "system"
```

要始终使用 Bun 的 shell 而不是系统 shell：

```toml
[run]
# Windows 上的默认值
shell = "bun"
```

### `run.bun` - 自动将 `node` 别名为 `bun`

当为 `true` 时，这会为所有由 `bun run` 或 `bun` 调用的脚本或可执行文件在 `$PATH` 前添加一个指向 `bun` 二进制文件的 `node` 符号链接。

这意味着如果您的脚本运行 `node`，它实际上会运行 `bun`，而不需要更改您的脚本。这是递归工作的，所以如果您的脚本运行另一个运行 `node` 的脚本，它也会运行 `bun`。这也适用于 shebang，所以如果您的脚本有一个指向 `node` 的 shebang，它实际上会运行 `bun`。

默认情况下，如果 `node` 不在您的 `$PATH` 中，则启用此功能。

```toml
[run]
# 相当于为所有 `bun run` 命令添加 `bun --bun`
bun = true
```

您可以通过运行以下命令来测试：

```sh
$ bun --bun which node # /path/to/bun
$ bun which node # /path/to/node
```

此选项相当于为所有 `bun run` 命令添加 `--bun` 前缀：

```sh
bun --bun run dev
bun --bun dev
bun run --bun dev
```

如果设置为 `false`，这将禁用 `node` 符号链接。

### `run.silent` - 抑制报告正在运行的命令

当为 `true` 时，抑制 `bun run` 或 `bun` 正在运行的命令的输出。

```toml
[run]
silent = true
```

没有此选项，正在运行的命令将打印到控制台：

```sh
$ bun run dev
> $ echo "Running \"dev\"..."
Running "dev"...
```

使用此选项，正在运行的命令不会打印到控制台：

```sh
$ bun run dev
Running "dev"...
```

这相当于为所有 `bun run` 命令传递 `--silent`：

```sh
bun --silent run dev
bun --silent dev
bun run --silent dev
```
