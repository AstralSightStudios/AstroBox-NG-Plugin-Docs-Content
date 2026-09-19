---
title: HTTP 服务器
---

`astrobox:psys-host-v4/http-server`，**Level 4 起可用**。

在设备本机起一个 HTTP 端口。最常见的用途是接 OAuth 回调（把
`http://127.0.0.1:<port>/callback` 当 redirect_uri），也可以给外部工具暴露小接口。

## 权限

| 权限 | 作用 |
|----|----|
| `http-server` | 起服务器（仅回环地址） |
| `http-server.lan` | 额外允许绑到所有网卡，局域网可见 |

## 插件要实现 handler

目标世界要换成 `psys-world-v4-http`，它在基础世界之上多一个导出：

```wit
world psys-world-v4-http {
  include psys-world-v4;
  export astrobox:psys-plugin-v4/http;
}
```

没实现这个导出就调用 `start`，会直接返回错误。

```rust
wit_bindgen::generate!({ path: "wit", world: "psys-world-v4-http", generate_all });

use crate::exports::astrobox::psys_plugin_v4::http;
use crate::astrobox::psys_host_v4::http_server;

impl http::Guest for Plugin {
    async fn handle(server_id: u32, request: http_server::Request) -> http_server::Response {
        http_server::Response {
            status: 200,
            headers: vec![http_server::Header {
                name: "content-type".into(),
                value: "text/plain; charset=utf-8".into(),
            }],
            body: format!("{} {}", request.method, request.path).into_bytes(),
        }
    }
}
```

## 接口

```wit
start: async func(options: server-options) -> result<server-info, string>;
stop: async func(id: u32) -> result<_, string>;
list-servers: func() -> list<server-info>;
```

`server-options.port` 填 `0` 表示让系统分配空闲端口，实际端口看返回的 `server-info`：

```rust
let info = http_server::start(http_server::ServerOptions {
    port: 0,
    bind_all_interfaces: false,
}).await?;

// info.url 形如 http://127.0.0.1:53124
```

## 行为与限制

- **默认只绑 `127.0.0.1`**。绑到所有网卡要另外申请 `http-server.lan`。
- **请求体上限 16 MiB**。handler 拿到的是完整 body（WIT 里是 `list<u8>`），不做流式。
- 监听在宿主侧，**生命周期绑死在插件实例上**：插件停止或热重载时端口一定会被释放。
- 多个服务器可以同时开，用 `handle` 的 `server-id` 区分。
