---
title: API Level
---

## 什么是 API Level

**API Level** 用于描述插件所使用的 **AstroBox 插件 API 等级**。  

插件在 `manifest.json` 中通过 `api_level` 字段声明自己所依赖的 API 等级：

```json
"api_level": 2
```

AstroBox 在加载插件时，会根据当前运行环境所支持的 API Level 决定插件是否可以正常运行。

---

## 兼容性原则

AstroBox 的 API Level 设计遵循以下原则：

### 1. 向下兼容

- **高版本 AstroBox 兼容低 API Level 的插件**
- 插件只要声明的 `api_level` ≤ 当前 AstroBox 支持的最高 API Level，即可被加载

例如：

| AstroBox 版本 | 支持的最高 API Level |
|-------------|--------------------|
| 2.0.0       | 2 |
| 2.1.0       | 3 |

- 使用 `api_level = 2` 的插件可以运行在 AstroBox 2.0.x 上  
- 使用 `api_level = 3` 的插件 **无法** 运行在 AstroBox 2.0.x 上  

---

### 2. 不保证向上兼容

- 插件 **不能假定** 更低版本的 AstroBox 支持更高的 API Level
- 如果插件声明的 API Level 高于当前 AstroBox 所支持的等级，插件将被拒绝加载

---

## API Level 与 AstroBox 版本对应关系

下表列出了目前已定义的 API Level 及其对应的 AstroBox 版本：

| API Level | 最低 AstroBox 版本 | 说明 |
|---------|------------------|----|
| 2 | CBT 1 / 2 | v2 第一次/第二次内测 |
| 3 | CBT 3 / 2.0.0 | v2 第三次内测/正式版 |

> ⚠️ 注意：AstroBox 只保证在**最低版本及以上**的版本中支持对应 API Level。

## API Level 升级说明

当 AstroBox 引入新的 API Level 时：

- 新增能力 **只会在更高 API Level 中提供**
- 旧 API Level 的行为保持不变
- WIT 文件中将加入新的接口，但插件作者需要 **主动升级 api_level** 才能使用新能力

升级 API Level 可能意味着：

- 需要调整插件代码
- 需要重新评估所需权限
- 可能影响插件可运行的 AstroBox 最低版本
