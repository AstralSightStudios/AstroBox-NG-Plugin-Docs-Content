---
title: Network
---
## 功能简介

实现发送网络请求。

## 权限声明

使用此接口需要向 manifest 的 permissions 中加入 `network` 权限。

## 相关类型

- **FetchOptions**
  - `method?: string` 默认为 GET
  - `headers?: Record<string, string>`
  - `body?: Uint8Array | string`
  - `raw?: boolean` 当此值为 true 时，Response 的 body 为 Uint8Array，否则为字符串（默认为 false）
- **FetchResponse**
  - `status: number`
  - `headers: Record<string, string>`
  - `contentType: string`
  - `body: Uint8Array|string` string 只支持 utf-8 编码

## 接口

### fetch

- **签名**：`fetch(url: string, options: FetchOptions): Promise<FetchResponse>`
- **说明**：与浏览器 `fetch` 类似。

## 示例

```typescript
import AstroBox from "astrobox-plugin-sdk";

const res = await AstroBox.network.fetch("https://example.com/api", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: 1 }),
});

console.log(res.status, res.contentType);
```
