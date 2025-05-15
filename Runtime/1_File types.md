Bun 打包器实现了一组默认的加载器。一般来说，打包器和运行时都默认支持相同的文件类型集。

`.js` `.cjs` `.mjs` `.mts` `.cts` `.ts` `.tsx` `.jsx` `.toml` `.json` `.txt` `.wasm` `.node` `.html`

Bun 使用文件扩展名来确定应该使用哪个内置_加载器_来解析文件。每个加载器都有一个名称，如 `js`、`tsx` 或 `json`。这些名称在构建[插件](https://bun.sh/docs/bundler/plugins)时使用，这些插件用自定义加载器扩展 Bun。

您可以使用 'loader' 导入属性显式指定要使用的加载器。

```ts
import my_toml from "./my_file" with { loader: "toml" };
```

## 内置加载器

### `js`

**JavaScript**。`.cjs` 和 `.mjs` 的默认加载器。

解析代码并应用一组默认转换，如死代码消除和树摇。注意，Bun 目前不会尝试向下转换语法。

### `jsx`

**JavaScript + JSX**。`.js` 和 `.jsx` 的默认加载器。

与 `js` 加载器相同，但支持 JSX 语法。默认情况下，JSX 被向下转换为普通 JavaScript；具体如何完成取决于您的 `tsconfig.json` 中的 `jsx*` 编译器选项。有关更多信息，请参阅 TypeScript 文档[关于 JSX](https://www.typescriptlang.org/docs/handbook/jsx.html)。

### `ts`

**TypeScript 加载器**。`.ts`、`.mts` 和 `.cts` 的默认加载器。

去除所有 TypeScript 语法，然后与 `js` 加载器行为相同。Bun 不执行类型检查。

### `tsx`

**TypeScript + JSX 加载器**。`.tsx` 的默认加载器。将 TypeScript 和 JSX 都转译为普通 JavaScript。

### `json`

**JSON 加载器**。`.json` 的默认加载器。

JSON 文件可以直接导入。

```ts
import pkg from "./package.json";
pkg.name; // => "my-package"
```

在打包期间，解析后的 JSON 作为 JavaScript 对象内联到包中。

```ts
var pkg = {
  name: "my-package",
  // ... 其他字段
};
pkg.name;
```

如果 `.json` 文件作为入口点传递给打包器，它将被转换为一个 `.js` 模块，该模块 `export default` 解析后的对象。

{% codetabs %}

```json#Input
{
  "name": "John Doe",
  "age": 35,
  "email": "johndoe@example.com"
}
```

```js#Output
export default {
  name: "John Doe",
  age: 35,
  email: "johndoe@example.com"
}
```

{% /codetabs %}

### `toml`

**TOML 加载器**。`.toml` 的默认加载器。

TOML 文件可以直接导入。Bun 将使用其快速的原生 TOML 解析器解析它们。

```ts
import config from "./bunfig.toml";
config.logLevel; // => "debug"

// 通过导入属性：
// import myCustomTOML from './my.config' with {type: "toml"};
```

在打包期间，解析后的 TOML 作为 JavaScript 对象内联到包中。

```ts
var config = {
  logLevel: "debug",
  // ...其他字段
};
config.logLevel;
```

如果 `.toml` 文件作为入口点传递，它将被转换为一个 `.js` 模块，该模块 `export default` 解析后的对象。

{% codetabs %}

```toml#Input
name = "John Doe"
age = 35
email = "johndoe@example.com"
```

```js#Output
export default {
  name: "John Doe",
  age: 35,
  email: "johndoe@example.com"
}
```

{% /codetabs %}

### `text`

**文本加载器**。`.txt` 的默认加载器。

文本文件的内容被读取并作为字符串内联到包中。
文本文件可以直接导入。文件被读取并作为字符串返回。

```ts
import contents from "./file.txt";
console.log(contents); // => "Hello, world!"

// 要将 html 文件作为文本导入
// 可以使用 "type" 属性覆盖默认加载器。
import html from "./index.html" with { type: "text" };
```

在构建期间引用时，内容作为字符串内联到包中。

```ts
var contents = `Hello, world!`;
console.log(contents);
```

如果 `.txt` 文件作为入口点传递，它将被转换为一个 `.js` 模块，该模块 `export default` 文件内容。

{% codetabs %}

```txt#Input
Hello, world!
```

```js#Output
export default "Hello, world!";
```

{% /codetabs %}

### `napi`

**原生插件加载器**。`.node` 的默认加载器。

在运行时，可以直接导入原生插件。

```ts
import addon from "./addon.node";
console.log(addon);
```

在打包器中，`.node` 文件使用 [`file`](#file) 加载器处理。

### `sqlite`

**SQLite 加载器**。`with { "type": "sqlite" }` 导入属性

在运行时和打包器中，可以直接导入 SQLite 数据库。这将使用 [`bun:sqlite`](https://bun.sh/docs/api/sqlite) 加载数据库。

```ts
import db from "./my.db" with { type: "sqlite" };
```

这仅在 `target` 为 `bun` 时支持。

默认情况下，数据库与包是外部的（这样您就可以使用在其他地方加载的数据库），因此磁盘上的数据库文件不会被捆绑到最终输出中。

您可以使用 `"embed"` 属性更改此行为：

```ts
// 将数据库嵌入到包中
import db from "./my.db" with { type: "sqlite", embed: "true" };
```

使用[独立可执行文件](https://bun.sh/docs/bundler/executables)时，数据库被嵌入到单文件可执行文件中。

否则，要嵌入的数据库会被复制到 `outdir` 中，文件名会被哈希处理。

### `html`

html 加载器处理 HTML 文件并打包任何引用的资源。它将：

- 打包并哈希引用的 JavaScript 文件（`<script src="...">`）
- 打包并哈希引用的 CSS 文件（`<link rel="stylesheet" href="...">`）
- 哈希引用的图片（`<img src="...">`）
- 保留外部 URL（默认情况下，任何以 `http://` 或 `https://` 开头的内容）

例如，给定这个 HTML 文件：

{% codetabs %}

```html#src/index.html
<!DOCTYPE html>
<html>
  <body>
    <img src="./image.jpg" alt="Local image">
    <img src="https://example.com/image.jpg" alt="External image">
    <script type="module" src="./script.js"></script>
  </body>
</html>
```

{% /codetabs %}

它将输出一个带有打包资源的新 HTML 文件：

{% codetabs %}

```html#dist/output.html
<!DOCTYPE html>
<html>
  <body>
    <img src="./image-HASHED.jpg" alt="Local image">
    <img src="https://example.com/image.jpg" alt="External image">
    <script type="module" src="./output-ALSO-HASHED.js"></script>
  </body>
</html>
```

{% /codetabs %}

在底层，它使用 [`lol-html`](https://github.com/cloudflare/lol-html) 将脚本和链接标签提取为入口点，并将其他资源作为外部资源。

目前，选择器列表是：

- `audio[src]`
- `iframe[src]`
- `img[src]`
- `img[srcset]`
- `link:not([rel~='stylesheet']):not([rel~='modulepreload']):not([rel~='manifest']):not([rel~='icon']):not([rel~='apple-touch-icon'])[href]`
- `link[as='font'][href], link[type^='font/'][href]`
- `link[as='image'][href]`
- `link[as='style'][href]`
- `link[as='video'][href], link[as='audio'][href]`
- `link[as='worker'][href]`
- `link[rel='icon'][href], link[rel='apple-touch-icon'][href]`
- `link[rel='manifest'][href]`
- `link[rel='stylesheet'][href]`
- `script[src]`
- `source[src]`
- `source[srcset]`
- `video[poster]`
- `video[src]`

### `sh` 加载器

**Bun Shell 加载器**。`.sh` 文件的默认加载器

此加载器用于解析 [Bun Shell](https://bun.sh/docs/runtime/shell) 脚本。它仅在启动 Bun 本身时受支持，因此在打包器或运行时中不可用。

```sh
$ bun run ./script.sh
```

### `file`

**文件加载器**。所有未识别文件类型的默认加载器。

文件加载器将导入解析为导入文件的_路径/URL_。它通常用于引用媒体或字体资源。

```ts#logo.ts
import logo from "./logo.svg";
console.log(logo);
```

_在运行时_，Bun 检查 `logo.svg` 文件是否存在，并将其转换为磁盘上 `logo.svg` 位置的绝对路径。

```bash
$ bun run logo.ts
/path/to/project/logo.svg
```

_在打包器中_，情况略有不同。文件按原样复制到 `outdir` 中，导入解析为指向复制文件的相对路径。

```ts#Output
var logo = "./logo.svg";
console.log(logo);
```

如果为 `publicPath` 指定了值，导入将使用该值作为前缀来构建绝对路径/URL。

{% table %}

- 公共路径
- 解析后的导入

---

- `""`（默认）
- `/logo.svg`

---

- `"/assets"`
- `/assets/logo.svg`

---

- `"https://cdn.example.com/"`
- `https://cdn.example.com/logo.svg`

{% /table %}

{% callout %}
复制文件的位置和文件名由 [`naming.asset`](https://bun.sh/docs/bundler#naming) 的值确定。
{% /callout %}
此加载器按原样复制到 `outdir` 中。复制文件的名称使用 `naming.asset` 的值确定。

{% details summary="修复 TypeScript 导入错误" %}
如果您使用 TypeScript，可能会遇到如下错误：

```ts
// TypeScript 错误
// Cannot find module './logo.svg' or its corresponding type declarations.
```

这可以通过在项目中的任何位置创建 `*.d.ts` 文件（任何名称都可以）来解决，内容如下：

```ts
declare module "*.svg" {
  const content: string;
  export default content;
}
```

这告诉 TypeScript 任何来自 `.svg` 的默认导入都应该被视为字符串。
{% /details %}
