---
title: 运行环境
---
AstroBox 通过 [Boa JS](https://boajs.dev/) 引擎为每个插件构建 JavaScript 运行时，每个插件的运行时是独立的，但可通过 [Event](../js-api/event) 接口互相通信。

## 语法支持

[Boa JS](https://boajs.dev/) 引擎已实现对 [ECMAScript 规范](https://ecma-international.org/publications-and-standards/standards/ecma-262/) 近乎完整的支持，并在 `test-262` 中达成了 90% 的通过率。在我们提供的模版中，我们将 ES2020 作为 TypeScript 的编译目标。AstroBox 提供的 JavaScript 运行时还支持以下扩展项：

1. [console](https://developer.mozilla.org/zh-CN/docs/Web/API/console)（仅 log warn error debug）
2. [setTimeout](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout)
3. [setInterval](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setInterval)

## 全局变量

AstroBox 提供的 JavaScript 运行时具有这些额外的全局变量：

| 变量名 | 变量内容 | 示例内容 |
| --- | --- | --- |
| RUNTIME | 运行环境标识 | AstroBox |
| RUNTIME_VERSION | 运行环境版本（AstroBox 版本） | 1.0 |
| PLUGIN_NAME | 插件名称 | ExamplePlugin |
| PLUGIN_PATH | 插件路径 | /Users/searchstars/Library/Application Support/moe.astralsight.astrobox/plugins/ExamplePlugin |
| PLUGIN_VERSION | 插件版本 | 1.0 |
