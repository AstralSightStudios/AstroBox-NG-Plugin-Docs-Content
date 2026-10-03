---
title: 项目结构
---
在创建完项目后，你将看到如下的文件结构：

```bash
.
├── README.md
├── manifest.json
├── node_modules
│   ├── ...
├── package.json
├── pnpm-lock.yaml
├── rspack.config.ts
├── icon.png
├── src
│   └── index.ts
└── tsconfig.json
```

- `manifest.json`: 插件配置文件，详见[插件配置](../manifest)，build 后会自动复制到 dist
- `icon.png`: 插件图标，build 后会自动复制到 dist
- `rspack.config.ts`: Rspack 配置文件，详见[官方文档](https://rspack.rs/zh/config/)
- `tsconfig.json`: TypeScript 语言支持配置文件，详见[官方文档](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
- `src/index.ts`: 插件入口 ts
