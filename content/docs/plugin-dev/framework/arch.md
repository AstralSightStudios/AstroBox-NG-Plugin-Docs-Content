---
title: 工作原理
---

## 简介
AstroBox v2 的插件系统基于 [WebAssembly System Interface (WASI)](https://wasi.dev/) 构建，并通过 [WIT Component](https://github.com/WebAssembly/component-model) 实现 **Host（宿主端）与 Plugin（插件端）之间的互操作接口**。
这两项技术都是近年才趋于成熟的新兴标准，我们非常自豪地率先在 AstroBox 中集成它们，并为其带来了 **卓越的跨平台兼容性**。

2025 年 12 月 22 日，openvela 微信公众号[发文](https://mp.weixin.qq.com/s/CV9iAkGPuqcLf_jhkXSUFQ)表示将使用 LWAC 为 JS 应用注入原生动力，该技术基于 WebAssembly，与 AstroBox v2 的插件系统技术栈高度相近。

AstroBox 使用 [wasmtime](https://github.com/bytecodealliance/wasmtime) 运行 WASI 插件，性能几乎可与原生代码相媲美（在 iOS 上由于缺乏 JIT 支持，性能可能略有下降）。

⚠️ 注意：这是 AstroBox v2 的插件文档，如果您正在寻找 AstroBox v1 的插件文档，请访问 [这里](../../plugin-v1)，但我们不建议继续为v1开发插件。

---

## 运行流程

1. **插件开发与编译**
   开发者可以使用任何支持 [wit-bindgen](https://github.com/bytecodealliance/wit-bindgen) 的语言编写 AstroBox 插件。
   编译完成后会生成一个 `.wasm` 文件，该文件即为插件的主要可执行体。

2. **插件加载**
   插件的 manifest 文件中需指定该 `.wasm` 文件为入口点。
   当 AstroBox 启动插件时，会通过 [wasmtime](https://github.com/bytecodealliance/wasmtime) 加载并运行它。

3. **AOT 预编译与缓存**
   在加载过程中，AstroBox 首先会将插件从 WebAssembly 预编译（AOT）为 `.cwasm` 文件。
   这一步可显著提升执行效率，并减少后续插件加载的启动时间。

4. **索引与版本管理**
   预编译完成后，系统会将插件的 **wasm 哈希值** 和 **engine ID** 记录到 `precompiled_index`（路径为 `<APP_DATA_DIR>/plugins/precompiled-index.json`）。
   当检测到原始 `.wasm` 文件发生变化时，AstroBox 会自动触发重新编译，以确保插件始终保持最新的运行状态。
