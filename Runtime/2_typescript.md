# TypeScript

要安装 Bun 内置 API 的 TypeScript 定义，请安装 `@types/bun`：

```bash
$ bun add -d @types/bun # 开发依赖
```

安装后，在 TypeScript 文件中引用 `Bun` 全局变量不应该产生错误。

## 建议的 `compilerOptions`

以下是 Bun 项目的建议 `compilerOptions`：

```json
{
  "compilerOptions": {
    // 启用所有严格类型检查选项
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,

    // 其他建议
    "module": "esnext",
    "moduleResolution": "bundler",
    "target": "esnext",
    "lib": ["esnext"],
    "types": ["bun-types"],

    // 启用装饰器
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,

    // 启用其他功能
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  }
}
```

这些选项分为几个主要类别：

### 环境设置

- `"module": "esnext"` - 使用最新的模块系统
- `"moduleResolution": "bundler"` - 使用 Bun 的模块解析
- `"target": "esnext"` - 使用最新的 JavaScript 特性
- `"lib": ["esnext"]` - 包含最新的 JavaScript API 类型
- `"types": ["bun-types"]` - 包含 Bun 的类型定义

### 打包器模式

- `"noEmit": true` - 不生成输出文件（Bun 会处理这个）
- `"isolatedModules": true` - 确保每个文件都可以独立编译
- `"jsx": "react-jsx"` - 启用 React JSX 支持

### 最佳实践

- `"strict": true` - 启用所有严格类型检查选项
- `"skipLibCheck": true` - 跳过声明文件的类型检查
- `"esModuleInterop": true` - 启用 CommonJS 和 ES 模块之间的互操作性
- `"allowSyntheticDefaultImports": true` - 允许从没有默认导出的模块中导入
- `"forceConsistentCasingInFileNames": true` - 确保文件名大小写一致
- `"resolveJsonModule": true` - 允许导入 JSON 文件

### 更严格的标志

- `"noImplicitAny": true` - 禁止隐式的 `any` 类型
- `"strictNullChecks": true` - 启用严格的空值检查
- `"strictFunctionTypes": true` - 启用严格的函数类型检查
- `"strictBindCallApply": true` - 启用严格的 `bind`、`call` 和 `apply` 检查
- `"strictPropertyInitialization": true` - 确保类属性被正确初始化
- `"noImplicitThis": true` - 禁止 `this` 的隐式 `any` 类型
- `"alwaysStrict": true` - 以严格模式解析并生成 "use strict"

### 装饰器

- `"experimentalDecorators": true` - 启用装饰器支持
- `"emitDecoratorMetadata": true` - 为装饰器启用元数据反射
