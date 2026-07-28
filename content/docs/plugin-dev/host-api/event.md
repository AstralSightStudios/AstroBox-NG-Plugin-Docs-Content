---
title: Event 接口
---

向其他已加载插件广播 `plugin-message` 事件。

## 接口定义

```wit
interface event {
    send-event: func(event-name: string, payload: string);
}
```

## 函数

### send-event

- 参数：
  - `event-name: string` 事件名。
  - `payload: string` 事件载荷字符串（通常为 JSON）。
- 返回：`()`。
- 说明：发送方自己不会收到这条广播；其他活跃插件会在 `on_event` 中以 `plugin-message` 收到一个 JSON 字符串，结构通常为 `{"eventName":"...","payload":"..."}`。

## 注意事项

- 这是异步广播，但 Host API 本身是同步返回，宿主内部会把实际派发放到后台任务里执行。
- 只有“已加载且未禁用”的其他插件会收到广播。
- 宿主会把 `event-name` 和 `payload` 再包一层 JSON 后转成 `plugin-message`，并不是把 `payload` 原样直接作为 `event-payload` 广播出去。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub fn notify_ready() {
    psys_host::event::send_event("plugin-ready", r#"{"ok":true}"#);
}
```
```go tab="Go"
package plugin

import event "astroboxplugin/bindings/astrobox_psys_host_event"

func notifyReady() {
	event.SendEvent("plugin-ready", `{"ok":true}`)
}
```
