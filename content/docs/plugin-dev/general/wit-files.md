---
title: WIT 文件
---

## 什么是 WIT 文件

**WIT（WebAssembly Interface Types）** 文件用于描述 **WebAssembly 组件的接口定义**，包括：

- 函数签名
- 数据结构
- 模块 / 世界（world）定义
- 组件之间的调用契约

在 AstroBox 插件体系中，WIT 文件用于定义：

- 插件可以调用的 AstroBox 接口
- AstroBox 可以回调插件的能力边界

我们将 WIT 文件统一放置在 [AstroBox-Plugin-WIT](https://github.com/AstralSightStudios/AstroBox-Plugin-WIT) 仓库中，并在不同语言的插件模板中作为按需更新的git submodule存在。**请确保在插件开发过程中始终使用最新的 WIT 文件。**

---

## 概念区分

- WIT 是 **接口规范**
- WASM 是 **实现**
- API Level 决定 **哪些接口可用**

插件通过 WIT 文件在编译期获得类型安全的接口绑定。

---

## 更新策略

### 核心原则

> **WIT 文件将持续更新以新增接口，但不会移除或破坏已有接口。**

因此：

- ✅ **低 API Level 和高 API Level 的插件项目** **都可以使用最新的 WIT 文件**
- ❌ **无需停留在某个旧版本的 WIT**

---

## WIT 与 API Level 的关系

### 1. 编译期行为

- 使用最新 WIT 文件时：
  - 所有接口在类型层面都是可见的
  - 编译器允许你调用这些接口

### 2. 运行期行为

- 插件声明的 `api_level` 决定 **运行时可用接口集合**
- **低 API Level 插件：**
  - 无法使用高 API Level 新增的接口
  - 即使代码能够成功编译，运行时仍会被拒绝或报错

---

## 典型示例

假设：

- API Level 3 新增接口 `psys_host::dialog::show_dialog`
- 插件声明：

```json
"api_level": 2
```

即使：

- 使用了最新的 WIT 文件
- 编译阶段没有错误

在运行时：

- 调用该接口将失败
- AstroBox 会根据 API Level 进行拦截

---

## 常见问题

#### Q: 为什么允许“能编译但不能用”
这种设计的目的在于：

- 保证 **单一、持续演进的 WIT 接口文件**
- 避免 WIT 文件碎片化、版本锁死
- 将兼容性判断集中到 **运行时 + Manifest**

API Level 是**能力声明**，而不是类型系统的一部分。
