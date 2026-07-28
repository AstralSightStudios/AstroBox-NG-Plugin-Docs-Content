---
title: Installer
---
## 功能简介

将第三方资源添加至统一安装队列，由 AstroBox 统一执行安装流程。

## 权限声明

使用此接口需要向 manifest 的 permissions 中加入 `installer` 权限。

## 接口

- **addThirdPartyAppToQueue(filePath: string): void** – 追加应用安装包。
- **addWatchFaceToQueue(filePath: string): void** – 追加表盘文件。
- **addFirmwareToQueue(filePath: string): void** – 追加固件包。

## 示例

```typescript
import AstroBox from "astrobox-plugin-sdk";

AstroBox.installer.addThirdPartyAppToQueue(`${PLUGIN_PATH}/test.rpk`);
```
