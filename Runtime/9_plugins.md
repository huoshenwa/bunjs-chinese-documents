Bun 提供了一个通用的插件 API，可用于扩展运行时和打包器。

插件拦截导入并执行自定义加载逻辑：读取文件、转译代码等。它们可用于添加对额外文件类型的支持，如 `.scss` 或 `.yaml`。在 Bun 打包器的上下文中，插件可用于实现框架级功能，如 CSS 提取、宏和客户端-服务器代码共置。

## 生命周期钩子

插件可以注册回调函数，在打包的生命周期的各个点运行：

- [`onStart()`](#onstart)：在打包器开始打包时运行一次
- [`onResolve()`](#onresolve)：在解析模块之前运行
- [`onLoad()`](#onload)：在加载模块之前运行
- [`onBeforeParse()`](#onbeforeparse)：在解析文件之前在解析器线程中运行零拷贝原生插件

### 参考

类型的大致概述（请参考 Bun 的 `bun.d.ts` 获取完整的类型定义）：

```ts
type PluginBuilder = {
  onStart(callback: () => void): void;
  onResolve: (
    args: { filter: RegExp; namespace?: string },
    callback: (args: { path: string; importer: string }) => {
      path: string;
      namespace?: string;
    } | void,
  ) => void;
  onLoad: (
    args: { filter: RegExp; namespace?: string },
    defer: () => Promise<void>,
    callback: (args: { path: string }) => {
      loader?: Loader;
      contents?: string;
      exports?: Record<string, any>;
    },
  ) => void;
  config: BuildConfig;
};

type Loader = "js" | "jsx" | "ts" | "tsx" | "css" | "json" | "toml";
```

## 用法

插件被定义为一个简单的 JavaScript 对象，包含 `name` 属性和 `setup` 函数。

```tsx#myPlugin.ts
import type { BunPlugin } from "bun";

const myPlugin: BunPlugin = {
  name: "Custom loader",
  setup(build) {
    // 实现
  },
};
```

这个插件可以在调用 `Bun.build` 时传入 `plugins` 数组。

```ts
await Bun.build({
  entrypoints: ["./app.ts"],
  outdir: "./out",
  plugins: [myPlugin],
});
```

## 插件生命周期

### 命名空间

`onLoad` 和 `onResolve` 接受一个可选的 `namespace` 字符串。什么是命名空间？

每个模块都有一个命名空间。命名空间用于在转译代码中为导入添加前缀；例如，一个带有 `filter: /\.yaml$/` 和 `namespace: "yaml:"` 的加载器会将 `./myfile.yaml` 的导入转换为 `yaml:./myfile.yaml`。

默认命名空间是 `"file"`，不需要指定它，例如：`import myModule from "./my-module.ts"` 与 `import myModule from "file:./my-module.ts"` 相同。

其他常见的命名空间有：

- `"bun"`：用于 Bun 特定的模块（例如 `"bun:test"`，`"bun:sqlite"`）
- `"node"`：用于 Node.js 模块（例如 `"node:fs"`，`"node:path"`）

### `onStart`

```ts
onStart(callback: () => void): Promise<void> | void;
```

注册一个回调函数，在打包器开始新的打包时运行。

```ts
import { plugin } from "bun";

plugin({
  name: "onStart example",

  setup(build) {
    build.onStart(() => {
      console.log("Bundle started!");
    });
  },
});
```

回调函数可以返回一个 `Promise`。在打包过程初始化后，打包器会等待所有 `onStart()` 回调完成后再继续。

例如：

```ts
const result = await Bun.build({
  entrypoints: ["./app.ts"],
  outdir: "./dist",
  sourcemap: "external",
  plugins: [
    {
      name: "Sleep for 10 seconds",
      setup(build) {
        build.onStart(async () => {
          await Bunlog.sleep(10_000);
        });
      },
    },
    {
      name: "Log bundle time to a file",
      setup(build) {
        build.onStart(async () => {
          const now = Date.now();
          await Bun.$`echo ${now} > bundle-time.txt`;
        });
      },
    },
  ],
});
```

在上面的例子中，Bun 将等待第一个 `onStart()`（睡眠 10 秒）完成，以及第二个 `onStart()`（将打包时间写入文件）完成。

注意，`onStart()` 回调（像其他生命周期回调一样）没有能力修改 `build.config` 对象。如果您想修改 `build.config`，您必须直接在 `setup()` 函数中进行。

### `onResolve`

```ts
onResolve(
  args: { filter: RegExp; namespace?: string },
  callback: (args: { path: string; importer: string }) => {
    path: string;
    namespace?: string;
  } | void,
): void;
```

要打包您的项目，Bun 会遍历项目中所有模块的依赖树。对于每个导入的模块，Bun 实际上必须找到并读取该模块。"查找"部分被称为"解析"模块。

`onResolve()` 插件生命周期回调允许您配置如何解析模块。

`onResolve()` 的第一个参数是一个带有 `filter` 和 [`namespace`](#what-is-a-namespace) 属性的对象。filter 是一个正则表达式，在导入字符串上运行。实际上，这些允许您过滤自定义解析逻辑将应用于哪些模块。

`onResolve()` 的第二个参数是一个回调函数，对于 Bun 找到的每个匹配 `filter` 和第一个参数中定义的 `namespace` 的模块导入运行。

回调函数接收匹配模块的 _path_ 作为输入。回调函数可以返回模块的 _新路径_。Bun 将读取 _新路径_ 的内容并将其解析为模块。

例如，将所有 `images/` 的导入重定向到 `./public/images/`：

```ts
import { plugin } from "bun";

plugin({
  name: "onResolve example",
  setup(build) {
    build.onResolve({ filter: /.*/, namespace: "file" }, args => {
      if (args.path.startsWith("images/")) {
        return {
          path: args.path.replace("images/", "./public/images/"),
        };
      }
    });
  },
});
```

### `onLoad`

```ts
onLoad(
  args: { filter: RegExp; namespace?: string },
  defer: () => Promise<void>,
  callback: (args: { path: string, importer: string, namespace: string, kind: ImportKind  }) => {
    loader?: Loader;
    contents?: string;
    exports?: Record<string, any>;
  },
): void;
```

在 Bun 的打包器解析模块后，它需要读取模块的内容并解析它。

`onLoad()` 插件生命周期回调允许您在 Bun 读取和解析模块之前修改模块的 _内容_。

像 `onResolve()` 一样，`onLoad()` 的第一个参数允许您过滤此 `onLoad()` 调用将应用于哪些模块。

`onLoad()` 的第二个参数是一个回调函数，对于每个匹配的模块，在 Bun 将模块内容加载到内存中 _之前_ 运行。

这个回调函数接收匹配模块的 _path_、模块的 _importer_（导入该模块的模块）、模块的 _namespace_ 和模块的 _kind_ 作为输入。

回调函数可以为模块返回新的 `contents` 字符串以及新的 `loader`。

例如：

```ts
import { plugin } from "bun";

const envPlugin: BunPlugin = {
  name: "env plugin",
  setup(build) {
    build.onLoad({ filter: /env/, namespace: "file" }, args => {
      return {
        contents: `export default ${JSON.stringify(process.env)}`,
        loader: "js",
      };
    });
  },
});

Bun.build({
  entrypoints: ["./app.ts"],
  outdir: "./dist",
  plugins: [envPlugin],
});

// import env from "env"
// env.FOO === "bar"
```

这个插件将把所有形式为 `import env from "env"` 的导入转换为一个导出当前环境变量的 JavaScript 模块。

#### `.defer()`

传递给 `onLoad` 回调的参数之一是 `defer` 函数。这个函数返回一个 `Promise`，当所有 _其他_ 模块都已加载时解析。

这允许您延迟执行 `onLoad` 回调，直到所有其他模块都已加载。

这对于返回依赖于其他模块的模块内容很有用。

##### 示例：跟踪和报告未使用的导出

```ts
import { plugin } from "bun";

plugin({
  name: "track imports",
  setup(build) {
    const transpiler = new Bun.Transpiler();

    let trackedImports: Record<string, number> = {};

    // 每个通过这个 onLoad 回调的模块
    // 都会在 `trackedImports` 中记录其导入
    build.onLoad({ filter: /\.ts/ }, async ({ path }) => {
      const contents = await Bun.file(path).arrayBuffer();

      const imports = transpiler.scanImports(contents);

      for (const i of imports) {
        trackedImports[i.path] = (trackedImports[i.path] || 0) + 1;
      }

      return undefined;
    });

    build.onLoad({ filter: /stats\.json/ }, async ({ defer }) => {
      // 等待所有文件加载完成，确保
      // 每个文件都通过上面的 `onLoad()` 函数
      // 并跟踪它们的导入
      await defer();

      // 发出包含每个导入统计信息的 JSON
      return {
        contents: `export default ${JSON.stringify(trackedImports)}`,
        loader: "json",
      };
    });
  },
});
```

注意，`.defer()` 函数目前有一个限制，即每个 `onLoad` 回调只能调用一次。

## 原生插件

Bun 的打包器如此之快的原因之一是它是用原生代码编写的，并利用多线程并行加载和解析模块。

然而，用 JavaScript 编写的插件的一个限制是 JavaScript 本身是单线程的。

原生插件是作为 [NAPI](https://bun.sh/docs/api/node-api) 模块编写的，可以在多个线程上运行。这允许原生插件比 JavaScript 插件运行得更快。

此外，原生插件可以跳过不必要的工作，如将字符串传递给 JavaScript 所需的 UTF-8 -> UTF-16 转换。

以下是可用于原生插件的生命周期钩子：

- [`onBeforeParse()`](#onbeforeparse)：在 Bun 的打包器解析文件之前在任何线程上调用。

原生插件是 NAPI 模块，将生命周期钩子作为 C ABI 函数暴露。

要创建原生插件，您必须导出一个与您想要实现的原生生命周期钩子签名匹配的 C ABI 函数。

### 在 Rust 中创建原生插件

原生插件是 NAPI 模块，将生命周期钩子作为 C ABI 函数暴露。

要创建原生插件，您必须导出一个与您想要实现的原生生命周期钩子签名匹配的 C ABI 函数。

```bash
bun add -g @napi-rs/cli
napi new
```

然后安装这个 crate：

```bash
cargo add bun-native-plugin
```

现在，在 `lib.rs` 文件中，我们将使用 `bun_native_plugin::bun` 过程宏来定义一个实现我们原生插件的函数。

这是一个实现 `onBeforeParse` 钩子的示例：

```rs
use bun_native_plugin::{define_bun_plugin, OnBeforeParse, bun, Result, anyhow, BunLoader};
use napi_derive::napi;

/// 定义插件及其名称
define_bun_plugin!("replace-foo-with-bar");

/// 这里我们将实现 `onBeforeParse`，代码将所有出现的
/// `foo` 替换为 `bar`。
///
/// 我们使用 #[bun] 宏来生成一些样板代码。
///
/// 函数的参数（`handle: &mut OnBeforeParse`）告诉
/// 宏这个函数实现了 `onBeforeParse` 钩子。
#[bun]
pub fn replace_foo_with_bar(handle: &mut OnBeforeParse) -> Result<()> {
  // 获取输入源代码。
  let input_source_code = handle.input_source_code()?;

  // 获取文件的 Loader
  let loader = handle.output_loader();


  let output_source_code = input_source_code.replace("foo", "bar");

  handle.set_output_source_code(output_source_code, BunLoader::BUN_LOADER_JSX);

  Ok(())
}
```

在 Bun.build() 中使用它：

```typescript
import myNativeAddon from "./my-native-addon";
Bun.build({
  entrypoints: ["./app.tsx"],
  plugins: [
    {
      name: "my-plugin",

      setup(build) {
        build.onBeforeParse(
          {
            namespace: "file",
            filter: "**/*.tsx",
          },
          {
            napiModule: myNativeAddon,
            symbol: "replace_foo_with_bar",
            // external: myNativeAddon.getSharedState()
          },
        );
      },
    },
  ],
});
```

### `onBeforeParse`

```ts
onBeforeParse(
  args: { filter: RegExp; namespace?: string },
  callback: { napiModule: NapiModule; symbol: string; external?: unknown },
): void;
```

这个生命周期回调在 Bun 的打包器解析文件之前立即运行。

作为输入，它接收文件的内容，并可以选择返回新的源代码。

这个回调可以从任何线程调用，因此 napi 模块实现必须是线程安全的。