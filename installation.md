Bun 作为单个可执行文件发布，没有依赖项，可以通过几种不同的方式安装。

## 安装

### macOS 和 Linux

{% callout %}
**Linux 用户** — 安装 Bun 需要 `unzip` 包。使用 `sudo apt install unzip` 安装 `unzip` 包。
强烈建议使用 5.6 或更高版本的内核，但最低要求是 5.1。使用 `uname -r` 检查内核版本。
{% /callout %}

{% codetabs %}

```bash#macOS/Linux_(curl)
$ curl -fsSL https://bun.sh/install | bash # 适用于 macOS、Linux 和 WSL
# 安装特定版本
$ curl -fsSL https://bun.sh/install | bash -s "bun-v$BUN_LATEST_VERSION"
```

```bash#npm
$ npm install -g bun # 您需要的最后一个 `npm` 命令
```

```bash#Homebrew
$ brew install oven-sh/bun/bun # 适用于 macOS 和 Linux
```

```bash#Docker
$ docker pull oven/bun
$ docker run --rm --init --ulimit memlock=-1:-1 oven/bun
```

{% /codetabs %}

### Windows

要安装，请在终端中粘贴以下内容：

{% codetabs %}

```powershell#PowerShell/cmd.exe
> powershell -c "irm bun.sh/install.ps1|iex"
```

```powershell#npm
> npm install -g bun # 您需要的最后一个 `npm` 命令
```

```powershell#Scoop
> scoop install bun
```

{% /codetabs %}

{% callout %}
Bun 需要 Windows 10 版本 1809 或更高版本
{% /callout %}

如需支持和讨论，请加入我们的 Discord 上的 [#windows 频道](http://bun.sh/discord)。

## Docker

Bun 提供了一个支持 Linux x64 和 arm64 的 [Docker 镜像](https://hub.docker.com/r/oven/bun/tags)。

```bash
$ docker pull oven/bun
$ docker run --rm --init --ulimit memlock=-1:-1 oven/bun
```

还有针对不同操作系统的镜像变体。

```bash
$ docker pull oven/bun:debian
$ docker pull oven/bun:slim
$ docker pull oven/bun:distroless
$ docker pull oven/bun:alpine
```

## 检查安装

要检查 Bun 是否安装成功，请打开一个新的终端窗口并运行 `bun --version`。

```sh
$ bun --version
1.x.y
```

要查看您正在使用的 [oven-sh/bun](https://github.com/oven-sh/bun) 的确切提交，请运行 `bun --revision`。

```sh
$ bun --revision
1.x.y+b7982ac13189
```

如果您已安装 Bun 但看到 `command not found` 错误，您可能需要手动将安装目录（`~/.bun/bin`）添加到您的 `PATH` 中。

### 如何添加 `PATH`

{% details summary="Linux / Mac" %}
首先，确定您使用的 shell：

```sh
$ echo $SHELL
/bin/zsh # 或 /bin/bash 或 /bin/fish
```

然后将以下行添加到您的 shell 配置文件的底部。

{% codetabs %}

```bash#~/.zshrc
# 添加到 ~/.zshrc
export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"
```

```bash#~/.bashrc
# 添加到 ~/.bashrc
export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"
```

```sh#~/.config/fish/config.fish
# 添加到 ~/.config/fish/config.fish
export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"
```

{% /codetabs %}
保存文件。您需要打开一个新的 shell/终端窗口才能使更改生效。

{% /details %}

{% details summary="Windows" %}
首先，确定 bun 二进制文件是否正确安装在您的系统上：

```pwsh
& "$env:USERPROFILE\.bun\bin\bun" --version
```

如果命令成功运行但 `bun --version` 不被识别，这意味着 bun 不在您系统的 `PATH` 中。要修复此问题，请打开 Powershell 终端并运行以下命令：

```pwsh
[System.Environment]::SetEnvironmentVariable(
    "Path",
    [System.Environment]::GetEnvironmentVariable("Path", "User") + ";$env:USERPROFILE\.bun\bin",
    [System.EnvironmentVariableTarget]::User
)
```

运行命令后，重启终端并使用 `bun --version` 测试

{% /details %}

## 升级

安装后，二进制文件可以自行升级。

```sh
$ bun upgrade
```

{% callout %}
**Homebrew 用户** — 为避免与 Homebrew 冲突，请改用 `brew upgrade bun`。

**Scoop 用户** — 为避免与 Scoop 冲突，请改用 `scoop update bun`。

{% /callout %}

## Canary 构建

Bun 在每次提交到 `main` 时都会自动发布一个（未经测试的）canary 构建。要升级到最新的 canary 构建：

```sh
$ bun upgrade --canary
```

canary 构建对于在稳定版本发布之前测试新功能和错误修复很有用。为了帮助 Bun 团队更快地修复错误，canary 构建会自动将崩溃报告上传到 Bun 团队。

[查看 canary 构建](https://github.com/oven-sh/bun/releases/tag/canary)

{% callout %}
**注意** — 要从 canary 切换回稳定版本，请运行 `bun upgrade --stable`。
{% /callout %}

## 安装旧版本的 Bun

由于 Bun 是单个二进制文件，您可以通过使用特定版本重新运行安装程序脚本来安装旧版本的 Bun。

### 在 Linux/Mac 上安装特定版本的 Bun

要安装特定版本的 Bun，您可以将您想要安装的版本的 git 标签传递给安装脚本，例如 `bun-v1.2.0` 或 `bun-v$BUN_LATEST_VERSION`。

```sh
$ curl -fsSL https://bun.sh/install | bash -s "bun-v$BUN_LATEST_VERSION"
```

### 在 Windows 上安装特定版本的 Bun

在 Windows 上，您可以通过将版本号传递给 Powershell 安装脚本来安装特定版本的 Bun。

```sh
# PowerShell:
$ iex "& {$(irm https://bun.sh/install.ps1)} -Version $BUN_LATEST_VERSION"
```

## 直接下载 Bun 二进制文件

要直接下载 Bun 二进制文件，您可以访问 GitHub 上的[发布页面](https://github.com/oven-sh/bun/releases)。

为了方便起见，这里是最新版本的下载链接：

- [`bun-linux-x64.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-linux-x64.zip)
- [`bun-linux-x64-baseline.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-linux-x64-baseline.zip)
- [`bun-linux-x64-musl.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-linux-x64-musl.zip)
- [`bun-linux-x64-musl-baseline.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-linux-x64-musl-baseline.zip)
- [`bun-windows-x64.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-windows-x64.zip)
- [`bun-windows-x64-baseline.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-windows-x64-baseline.zip)
- [`bun-darwin-aarch64.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-darwin-aarch64.zip)
- [`bun-linux-aarch64.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-linux-aarch64.zip)
- [`bun-linux-aarch64-musl.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-linux-aarch64-musl.zip)
- [`bun-darwin-x64.zip`](https://github.com/oven-sh/bun/releases/latest/download/bun-darwin-x64.zip)

`musl` 二进制文件是为默认不提供 glibc 库的发行版构建的，而是依赖 musl。两个最流行的发行版是 Void Linux 和 Alpine Linux，后者在 Docker 容器中大量使用。如果您遇到类似以下错误：`bun: /lib/x86_64-linux-gnu/libm.so.6: version GLIBC_2.29' not found (required by bun)`，请尝试使用 musl 二进制文件。Bun 的安装脚本会自动为您的系统选择正确的二进制文件。

Bun 的 `x64` 二进制文件针对 Haswell CPU 架构，这意味着它们需要 AVX 和 AVX2 指令。对于 Linux 和 Windows，还提供了针对 Nehalem 架构的 `x64-baseline` 二进制文件。如果您在运行 Bun 时遇到"Illegal Instruction"错误，请尝试使用 `baseline` 二进制文件。Bun 的安装脚本会自动为您的系统选择正确的二进制文件，这有助于避免此问题。Baseline 构建比常规构建慢，所以只在必要时使用它们。

Bun 还发布 `darwin-x64-baseline` 二进制文件，但这些只是 `darwin-x64` 的副本，所以它们仍然具有相同的 CPU 要求。我们只维护这些是因为一些工具期望它们存在。Bun 需要 macOS 13.0 或更高版本，它不支持任何不满足我们要求的 CPU。

## 卸载

如果您需要从系统中删除 Bun，请使用以下命令。

{% codetabs %}

```bash#macOS/Linux_(curl)
$ rm -rf ~/.bun # 适用于 macOS、Linux 和 WSL
```

```powershell#Windows
> powershell -c ~\.bun\uninstall.ps1
```

```powershell#Scoop
> scoop uninstall bun
```

```bash#npm
$ npm uninstall -g bun
```

```bash#Homebrew
$ brew uninstall bun
```

{% /codetabs %}
