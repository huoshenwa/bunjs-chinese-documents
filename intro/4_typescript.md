要安装 Bun 内置 API 的 TypeScript 定义，请安装 `@types/bun`。

```sh
$ bun add -d @types/bun # 开发依赖
```

此时，您应该能够在 TypeScript 文件中引用 `Bun` 全局变量，而不会在编辑器中看到错误。

```ts
console.log(Bun.version);
```

## 建议的 `compilerOptions`

Bun 支持顶级 await、JSX 和带扩展名的 `.ts` 导入等功能，这些在 TypeScript 中默认是不允许的。以下是一组推荐的 Bun 项目 `compilerOptions`，这样您就可以使用这些功能，而不会看到 TypeScript 的编译器警告。

```jsonc
{
  "compilerOptions": {
    // 环境设置和最新特性
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "Preserve",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,

    // 打包器模式
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,

    // 最佳实践
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,

    // 一些更严格的标志（默认禁用）
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noPropertyAccessFromIndexSignature": false,
  },
}
```

如果您在新目录中运行 `bun init`，这个 `tsconfig.json` 将为您生成。（更严格的标志默认是禁用的。）

```sh
$ bun init
```
