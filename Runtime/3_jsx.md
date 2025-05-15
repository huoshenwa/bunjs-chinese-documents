Bun 默认支持 `.jsx` 和 `.tsx` 文件。Bun 的内部转译器在执行前将 JSX 语法转换为普通 JavaScript。

```tsx#react.tsx
function Component(props: {message: string}) {
  return (
    <body>
      <h1 style={{color: 'red'}}>{props.message}</h1>
    </body>
  );
}

console.log(<Component message="Hello world!" />);
```

## 配置

Bun 读取您的 `tsconfig.json` 或 `jsconfig.json` 配置文件来确定如何在内部执行 JSX 转换。为了避免使用这些文件，以下选项也可以在 [`bunfig.toml`](https://bun.sh/docs/runtime/bunfig) 中定义。

以下编译器选项会被遵循。

### [`jsx`](https://www.typescriptlang.org/tsconfig#jsx)

JSX 结构如何在内部转换为普通 JavaScript。下表列出了 `jsx` 的可能值，以及它们对以下简单 JSX 组件的转译：

```tsx
<Box width={5}>Hello</Box>
```

{% table %}

- 编译器选项
- 转译输出

---

- ```json
  {
    "jsx": "react"
  }
  ```

- ```tsx
  import { createElement } from "react";
  createElement("Box", { width: 5 }, "Hello");
  ```

---

- ```json
  {
    "jsx": "react-jsx"
  }
  ```

- ```tsx
  import { jsx } from "react/jsx-runtime";
  jsx("Box", { width: 5 }, "Hello");
  ```

---

- ```json
  {
    "jsx": "react-jsxdev"
  }
  ```

- ```tsx
  import { jsxDEV } from "react/jsx-dev-runtime";
  jsxDEV(
    "Box",
    { width: 5, children: "Hello" },
    undefined,
    false,
    undefined,
    this,
  );
  ```

  `jsxDEV` 变量名是 React 使用的约定。`DEV` 后缀是一种可见的方式，表明代码旨在用于开发。React 的开发版本较慢，并包含额外的有效性检查和调试工具。

---

- ```json
  {
    "jsx": "preserve"
  }
  ```

- ```tsx
  // JSX 不会被转译
  // Bun 目前不支持 "preserve"
  <Box width={5}>Hello</Box>
  ```

{% /table %}

### [`jsxFactory`](https://www.typescriptlang.org/tsconfig#jsxFactory)

{% callout %}
**注意** — 仅当 `jsx` 为 `react` 时适用。
{% /callout %}

用于表示 JSX 结构的函数名。默认值为 `"createElement"`。这对于像 [Preact](https://preactjs.com/) 这样使用不同函数名（`"h"`）的库很有用。

{% table %}

- 编译器选项
- 转译输出

---

- ```json
  {
    "jsx": "react",
    "jsxFactory": "h"
  }
  ```

- ```tsx
  import { h } from "react";
  h("Box", { width: 5 }, "Hello");
  ```

{% /table %}

### [`jsxFragmentFactory`](https://www.typescriptlang.org/tsconfig#jsxFragmentFactory)

{% callout %}
**注意** — 仅当 `jsx` 为 `react` 时适用。
{% /callout %}

用于表示 [JSX 片段](https://react.dev/reference/react/Fragment)的函数名，如 `<>Hello</>`；仅当 `jsx` 为 `react` 时适用。默认值为 `"Fragment"`。

{% table %}

- 编译器选项
- 转译输出

---

- ```json
  {
    "jsx": "react",
    "jsxFactory": "myjsx",
    "jsxFragmentFactory": "MyFragment"
  }
  ```

- ```tsx
  // 输入
  <>Hello</>;

  // 输出
  import { myjsx, MyFragment } from "react";
  myjsx(MyFragment, null, "Hello");
  ```

{% /table %}

### [`jsxImportSource`](https://www.typescriptlang.org/tsconfig#jsxImportSource)

{% callout %}
**注意** — 仅当 `jsx` 为 `react-jsx` 或 `react-jsxdev` 时适用。
{% /callout %}

组件工厂函数（`createElement`、`jsx`、`jsxDEV` 等）将从其导入的模块。默认值为 `"react"`。在使用像 Preact 这样的组件库时，这通常是必要的。

{% table %}

- 编译器选项
- 转译输出

---

- ```jsonc
  {
    "jsx": "react",
    // jsxImportSource 未定义
    // 默认为 "react"
  }
  ```

- ```tsx
  import { jsx } from "react/jsx-runtime";
  jsx("Box", { width: 5, children: "Hello" });
  ```

---

- ```jsonc
  {
    "jsx": "react-jsx",
    "jsxImportSource": "preact",
  }
  ```

- ```tsx
  import { jsx } from "preact/jsx-runtime";
  jsx("Box", { width: 5, children: "Hello" });
  ```

---

- ```jsonc
  {
    "jsx": "react-jsxdev",
    "jsxImportSource": "preact",
  }
  ```

- ```tsx
  // 自动附加 /jsx-runtime
  import { jsxDEV } from "preact/jsx-dev-runtime";
  jsxDEV(
    "Box",
    { width: 5, children: "Hello" },
    undefined,
    false,
    undefined,
    this,
  );
  ```

{% /table %}

### JSX 编译指示

所有这些值都可以使用_编译指示_在每个文件的基础上设置。编译指示是一个特殊注释，用于在特定文件中设置编译器选项。

{% table %}

- 编译指示
- 等效配置

---

- ```ts
  // @jsx h
  ```

- ```jsonc
  {
    "jsxFactory": "h",
  }
  ```

---

- ```ts
  // @jsxFrag MyFragment
  ```
- ```jsonc
  {
    "jsxFragmentFactory": "MyFragment",
  }
  ```

---

- ```ts
  // @jsxImportSource preact
  ```
- ```jsonc
  {
    "jsxImportSource": "preact",
  }
  ```

{% /table %}

## 日志记录

Bun 为 JSX 实现了特殊的日志记录，以使调试更容易。给定以下文件：

```tsx#index.tsx
import { Stack, UserCard } from "./components";

console.log(
  <Stack>
    <UserCard name="Dom" bio="Street racer and Corona lover" />
    <UserCard name="Jakob" bio="Super spy and Dom's secret brother" />
  </Stack>
);
```

Bun 将在记录时美化打印组件树：

{% image src="https://github.com/oven-sh/bun/assets/3084745/d29db51d-6837-44e2-b8be-84fc1b9e9d97" / %}

## 属性简写

Bun 运行时还支持 JSX 的"属性简写"。这是一种有用的简写语法，用于将变量分配给具有相同名称的属性。

```tsx
function Div(props: {className: string;}) {
  const {className} = props;

  // 不使用简写
  return <div className={className} />;
  // 使用简写
  return <div {className} />;
}
```
