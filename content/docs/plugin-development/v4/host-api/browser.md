---
title: 应用内浏览器
---

`astrobox:psys-host-v4/browser`，**Level 4 起可用**。

打开一个**可控**的 WebView：能拦截导航、读 cookie、注入 JS。主要用途是复刻第三方
应用的登录流程——打开对方的 OAuth 授权页，把回调地址登记成拦截前缀，回调发生时
宿主**取消这次导航**并把完整 URL 交还插件，而不是让系统去唤起那个 App。

## 权限

需要 `browser`。这个能力能看到用户在页面里输入的一切（包括第三方账号密码），
因此**每次 `open` 都会走一次授权**。

## 劫持 OAuth 回调

```rust
use crate::astrobox::psys_host_v4::browser;

let id = browser::open(browser::OpenOptions {
    url: "https://example.com/oauth/authorize?client_id=demo".into(),
    title: Some("登录".into()),
    user_agent: None,
    // 对方 App 的私有回调 scheme，命中即取消导航
    intercept_prefixes: vec!["demoapp://oauth".into()],
    close_on_intercept: true,
    ephemeral: true,
    width: None,
    height: None,
}).await?;

// 阻塞直到回调被拦下来，拿到带 code/state 的完整 URL
let callback_url = browser::wait_for_intercept(id, Some(120_000)).await?;
```

`intercept-prefixes` 按**字符串前缀**匹配，自定义 scheme 直接写 `demoapp://` 即可。
命中后系统不会去唤起真正的那个 App，参数留在插件手里。

## 其余接口

```wit
navigate: async func(id: u32, url: string) -> result<_, string>;
current-url: async func(id: u32) -> result<string, string>;
eval: async func(id: u32, script: string) -> result<string, string>;
get-cookies: async func(id: u32, url: string) -> result<list<cookie>, string>;
close: async func(id: u32) -> result<_, string>;
```

`eval` 返回 JSON 序列化后的结果。

## 平台差异

| 能力 | 桌面 | iOS | Android |
|----|----|----|----|
| 导航拦截 | ✅ | ✅ | ✅ |
| 读 cookie | ✅ | ✅ | ⚠️ 只有 name/value |
| `eval` | ✅ | ✅ | ✅ |
| `ephemeral` 隔离 | ✅ incognito | ✅ 非持久数据区 | ⚠️ 尽力而为 |

- 桌面用 Tauri 子 webview 窗口，iOS 用 WKWebView，Android 用 Dialog 里的 WebView。
- Android 的 `CookieManager` 是进程级共享的，`get-cookies` 只能拿到 `name`/`value`，
  `domain`/`path` 按请求 URL 回填；`ephemeral` 也无法做到真正隔离。
- iOS 上本插件另有一组 SFSafariViewController（供 App 自身登录），那套**拦不到导航
  也读不到 cookie**，与这里是两回事。

## 注意

- `wait-for-intercept` 在浏览器被关闭时返回错误；若**关闭前已经命中过拦截**，
  返回的仍是那个 URL（`close-on-intercept` 是成功路径，不算失败）。
- 用户手动关窗同样会唤醒等待中的调用，不会一直挂着。
