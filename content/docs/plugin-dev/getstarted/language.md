---
title: 语言选择
---

## 语言支持
理论上，只要语言支持 [wit-bindgen](https://github.com/bytecodealliance/wit-bindgen)，就可以用于开发 AstroBox v2 插件。
然而，我们**官方推荐并主要支持**以下三种语言：

- [**Rust**](./rust)（强烈推荐）
- **Go**
- **JavaScript** (暂不可用，需等待上游新增异步支持)

这些语言在生态、编译工具链和性能方面都已与 AstroBox 进行了充分适配和测试。

---

## 其他可用语言
除上述语言外，以下语言也具备一定程度的兼容性，并可通过 wit-bindgen 开发插件（但暂未提供官方支持或测试保障）：

- Java
- C / C++
- MoonBit
- Zig
- Python
- Ruby
- C#

---

> 对于追求性能、安全性与长期维护的开发者，推荐优先选择 **Rust**。
> 其对 WASI 与组件模型（Component Model）的支持最为完整，并且拥有最优的执行效率与错误检测机制。
