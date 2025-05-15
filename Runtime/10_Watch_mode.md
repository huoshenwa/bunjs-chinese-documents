Bun 通过 CLI 标志支持两种自动重载模式：

- `--watch` 模式，当导入的文件发生变化时，会硬重启 Bun 的进程。
- `--hot` 模式，当导入的文件发生变化时，会软重载代码（不重启进程）。

## `--watch` 模式

监视模式可以与 `bun test` 一起使用，或者在运行 TypeScript、JSX 和 JavaScript 文件时使用。

要在 `--watch` 模式下运行文件：

```bash
$ bun --watch index.tsx
```

要在 `--watch` 模式下运行测试：

```bash
$ bun --watch test
```

在 `--watch` 模式下，Bun 会跟踪所有导入的文件并监视它们的变化。当检测到变化时，Bun 会重启进程，保留初始运行时使用的相同 CLI 参数和环境变量。如果 Bun 崩溃，`--watch` 将尝试自动重启进程。

{% callout %}

**⚡️ 重载速度很快。** 您可能习惯的文件系统监视器有几层库包装原生 API，或者更糟的是，依赖于轮询。

相反，Bun 使用操作系统原生的文件系统监视器 API，如 kqueue 或 inotify 来检测文件变化。Bun 还进行了一些优化，使其能够扩展到更大的项目（例如设置较高的文件描述符 rlimit，静态分配文件路径缓冲区，尽可能重用文件描述符等）。

{% /callout %}

以下示例展示了 Bun 在编辑文件时实时重载，VSCode 配置为[每次按键时保存](https://code.visualstudio.com/docs/editor/codebasics#_save-auto-save)。

{% codetabs %}

```bash
$ bun run --watch watchy.tsx
```

```tsx#watchy.tsx
import { serve } from "bun";
console.log("I restarted at:", Date.now());

serve({
  port: 4003,

  fetch(request) {
    return new Response("Sup");
  },
});
```

{% /codetabs %}

在这个例子中，Bun 是
![bun watch gif](https://user-images.githubusercontent.com/709451/228439002-7b9fad11-0db2-4e48-b82d-2b88c8625625.gif)

在监视模式下运行 `bun test` 并启用 `save-on-keypress`：

```bash
$ bun --watch test
```

![bun test gif](https://user-images.githubusercontent.com/709451/228396976-38a23864-4a1d-4c96-87cc-04e5181bf459.gif)

{% callout %}

**`--no-clear-screen`** 标志在您不希望终端清除的场景中很有用，例如当使用 `concurrently` 等工具同时运行多个 `bun build --watch` 命令时。如果没有这个标志，一个实例的输出可能会清除其他实例的输出，可能会将一个实例的错误隐藏在另一个实例的输出之下。`--no-clear-screen` 标志类似于 TypeScript 的 `--preserveWatchOutput`，可以防止这个问题。它可以与 `--watch` 结合使用，例如：`bun build --watch --no-clear-screen`。

{% /callout %}

## `--hot` 模式

使用 `bun --hot` 在 Bun 执行代码时启用热重载。这与 `--watch` 模式不同，因为 Bun 不会硬重启整个进程。相反，它会检测代码变化并更新其内部模块缓存中的新代码。

**注意** — 这与浏览器中的热重载不同！许多框架提供了"热重载"体验，您可以编辑和保存前端代码（比如 React 组件），并在不刷新页面的情况下在浏览器中看到变化。Bun 的 `--hot` 是这种体验的服务器端等效。要在浏览器中获得热重载，请使用 [Vite](https://vitejs.dev) 等框架。

```bash
$ bun --hot server.ts
```

从入口点（上例中的 `server.ts`）开始，Bun 构建所有导入的源文件（不包括 `node_modules` 中的文件）的注册表，并监视它们的变化。当检测到变化时，Bun 执行"软重载"。所有文件都会重新评估，但所有全局状态（特别是 `globalThis` 对象）都会保留。

```ts#server.ts
// 让 TypeScript 满意
declare global {
  var count: number;
}

globalThis.count ??= 0;
console.log(`Reloaded ${globalThis.count} times`);
globalThis.count++;

// 防止 `bun run` 退出
setInterval(function () {}, 1000000);
```

如果您使用 `bun --hot server.ts` 运行此文件，每次保存文件时都会看到重载计数增加。

```bash
$ bun --hot index.ts
Reloaded 1 times
Reloaded 2 times
Reloaded 3 times
```

传统的文件监视器如 `nodemon` 会重启整个进程，因此 HTTP 服务器和其他有状态的对象会丢失。相比之下，`bun --hot` 能够在不停机的情况下反映更新的代码。

### HTTP 服务器

这使得例如更新 HTTP 请求处理程序而不关闭服务器本身成为可能。当您保存文件时，您的 HTTP 服务器将使用更新的代码重载，而不会重启进程。这导致重载速度非常快。

```ts#server.ts
globalThis.count ??= 0;
globalThis.count++;

Bun.serve({
  fetch(req: Request) {
    return new Response(`Reloaded ${globalThis.count} times`);
  },
  port: 3000,
});
```

{% callout %}
**注意** — 在 Bun 的未来版本中，计划支持 Vite 的 `import.meta.hot`，以实现更好的热重载生命周期管理，并与生态系统保持一致。

{% /callout %}

{% details summary="实现细节" %}

在热重载时，Bun：

- 重置内部 `require` 缓存和 ES 模块注册表（`Loader.registry`）
- 同步运行垃圾收集器（以最小化内存泄漏，以运行时性能为代价）
- 从头重新转译所有代码（包括源码映射）
- 使用 JavaScriptCore 重新评估代码

这个实现并不是特别优化。它会重新转译没有变化的文件。它没有尝试增量编译。这是一个起点。

{% /details %}
