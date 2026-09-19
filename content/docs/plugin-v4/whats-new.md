---
title: Level 4 有什么新东西
---

## 1. 接口全部是 `async func`

Level 2/3 的宿主接口长这样：

```wit
arch: func() -> future<string>;
```

插件侧要配合一整套 `wit_future::new()` + `wit_bindgen::spawn` 的样板，还得小心
「spawn 出去的任务没人 drain」这类坑。

Level 4 直接是：

```wit
arch: async func() -> string;
```

Rust 侧就是普通的 `.await`：

```rust
let arch = os::arch().await;
```

`on-load` 也是 `async func`，**可以直接 await 宿主接口**，不必再 `block_on`。

### 为什么必须这么改

组件模型把 `async` 记在**函数类型**上，宿主运行时据此决定一个任务是否允许阻塞
（wasmtime 的规则是 `may_block = 函数类型是 async || 已经返回`）。如果导出声明成普通
`func`，插件在里面 await 任何宿主调用都会被直接 trap 掉（`CannotBlockSyncTask`）。

反过来，只要类型是 `async`，即使某些语言的工具链只会做同步 lift/lower，也**依然被
允许阻塞等待宿主返回**——这正是让更多语言能接进来的前提。

## 2. WASI Preview 3

Level 4 插件运行在 WASI p3 环境里。宿主同时提供 p2 接口，因为大多数语言的运行时
（wasi-libc、CPython 等）目前仍按 p2 链接标准库，两边都装齐才不会缺导入。

manifest 里声明：

```json
"wasi_version": 3,
"api_level": 4
```

## 3. 双运行时共存

Level 2/3 的插件继续跑在原来的 wasmtime 上，Level 4 用新版，两者在同一个进程内共存。
升级 Level 4 **不会**影响已有插件的行为。

## 4. 错误信息不再丢失

Level 2/3 里失败是裸 `result`（Rust 侧 `Result<(), ()>`），拿不到原因。Level 4 统一成
`result<_, string>`：

```rust
match transport::send(&addr, &data).await {
    Ok(()) => {}
    Err(reason) => tracing::warn!("send failed: {reason}"),
}
```

## 5. 新增能力

- [HTTP 服务器](./host-api/http-server)：在本机起一个端口，常用于接 OAuth 回调
- [应用内浏览器](./host-api/browser)：可拦截导航、读 cookie、注入 JS

- [设备通知与实时活动](./host-api/notification)：向穿戴设备发送、更新及撤回通知和超级岛实时活动
- [宿主标识与账号资料](./host-api/identity)：读取宿主标识和已登录账号的脱敏绑定资料
