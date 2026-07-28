---
title: 运行环境
---

## 接口
AstroBox v2 的插件运行环境启用了对 [wasi-p2](https://wasi.dev/interfaces#wasi-02) 规范的支持，覆盖了当前所有已实现的稳定 WASI p2 特性，包括标准文件系统、日期时间以及系统级随机数（RNG）访问等功能。

此外，我们还集成了 `wasmtime-wasi-http` 模块，开发者可以在插件中使用 [waki](https://docs.rs/waki) 库，以类似 `reqwest` 的方式发送 HTTP 网络请求。

同时，AstroBox 宿主（Host）自身也暴露了大量接口供插件调用；相应地，插件也需要实现部分接口以供宿主访问。
详细的接口定义与实现规范可参考 **AstroBox API 文档**。

---

## 引擎配置
AstroBox v2 的插件执行引擎具备以下特性：

- **执行模式**
  - Windows / macOS / Linux / Android：使用 **Cranelift 引擎** 以 **JIT 模式** 执行。
  - iOS：使用 **Pulley64 引擎** 以 **解释模式（Interpreter）** 执行。

- **资源与功能限制**
  - 每个插件的最大内存占用：**128 MiB**
  - 启用特性：
    - `memory_may_move`
    - `wasm_component_model`
    - `wasm_component_model_async`
    - `async_support`
  - 禁用特性：
    - `wasm_memory64`

- **文件系统安全策略**

  出于安全性考虑，标准文件系统（std fs）接口仅允许插件访问自身目录下的文件。
  任何越界访问都会被立即拒绝并抛出错误。
  若确需访问其他路径，可通过宿主提供的安全接口，在 **用户明确授权** 的前提下实现访问。
