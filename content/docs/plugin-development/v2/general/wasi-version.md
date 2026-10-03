---
title: WASI 版本
---

## 什么是 WASI

**WASI（WebAssembly System Interface）** 是一套为 WebAssembly 提供的系统接口标准，用于在不依赖特定宿主操作系统的情况下，安全地访问诸如文件系统、时间、随机数、网络等系统能力。

在 AstroBox 插件体系中：

- 插件以 **WebAssembly（WASM）** 形式运行
- WASI 负责定义插件与宿主环境之间的系统调用边界
- `wasi_version` 用于声明插件所依赖的 WASI 接口版本

---

## 为什么需要 WASI Version

随着 WASI 标准的演进：

- 新的系统接口会被引入
- 旧接口可能被弃用或行为调整
- Rust / LLVM 等工具链的 WASI 支持也会随之变化

因此，AstroBox 需要通过 `wasi_version` 明确插件的运行时预期，以保证：

- 插件可以在合适的 WASI 运行环境中执行
- 不同 WASI 版本之间不会产生不可预期的行为差异

---

## Manifest 中的表示

在插件的 `manifest.json` 中：

```json
"wasi_version": 2
```

表示：

- 插件基于 **WASI Preview 2**（或等价稳定子集）构建
- AstroBox 将以对应版本的 WASI 运行环境加载该插件

> AstroBox 当前只支持 **WASI Preview 2**（或等价稳定子集）。

---

## WASI Version 与 Rust Target 的关系

在 Rust 中，WASI 版本通常通过 **编译目标（target）** 体现。

常见的 WASI Rust Target 包括：

| Rust Target | 对应 WASI 版本 | 说明 |
|------------|---------------|----|
| `wasm32-wasip2` | WASI Preview 2 | 当前的稳定 WASI 组件模型 |
| `wasm32-wasip3` | WASI Preview 3 | 新一代 WASI 组件模型，暂未稳定 |

---

## WASI 与 AstroBox API 的区别

需要特别区分：

| 项目 | 作用 |
|----|----|
| WASI | 提供基础系统接口（文件、时间、IO 等） |
| AstroBox API | 提供宿主平台能力（插件通信、UI、设备管理等） |

- WASI 是 **通用标准**
- AstroBox API 是 **平台私有能力**
- `wasi_version` 与 `api_level` **彼此独立，但共同决定插件运行环境**
