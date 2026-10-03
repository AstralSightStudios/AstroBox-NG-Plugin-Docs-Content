---
title: Register 接口
---

用于向宿主注册插件能力，例如消息接收、Deeplink、Provider 和设备详情页卡片。

## 接口定义

```wit
interface register {
    record transport-recv-filer {
        xiaomi-vela-v5-channel-id: u32,
        xiaomi-vela-v5-protobuf-typeid: u32,
    }

    enum provider-type {
        URL,
        CUSTOM
    }

    enum card-type {
        ELEMENT,
        TEXT
    }

    register-transport-recv: func(addr: string, filter: transport-recv-filer) -> future<result>;
    register-interconnect-recv: func(addr: string, pkg-name: string) -> future<result>;
    register-deeplink-action: func() -> future<result>;
    register-provider: func(name: string, provider-type: provider-type) -> future<result>;
    register-card: func(card-type: card-type, id: string, name: string) -> future<result>;
}
```

## 类型

### transport-recv-filer

- 这是 WIT 中的原始名字，虽然看起来像拼写错误，但当前接口名就是 `transport-recv-filer`，生成绑定里通常也会是 `TransportRecvFiler`。
- `xiaomi-vela-v5-channel-id`：Vela V5 通道 ID。
- `xiaomi-vela-v5-protobuf-typeid`：Vela V5 Protobuf 类型 ID。

### provider-type

- `URL`：URL 型 Provider。
- `CUSTOM`：自定义 Provider。

### card-type

- `ELEMENT`：元素卡片，与 `ui::render(id, el)` 或 `ui-v3::render(id, el)` 配合。
- `TEXT`：纯文本卡片，与 `ui::render-to-text-card` 配合。

## 函数

### register-transport-recv

根据过滤条件订阅设备端发送来的消息。注册成功后，插件会在 `on_event` 中收到 `transport-packet` 事件。

- 参数：
  - `addr: string` 设备地址。
  - `filter: transport-recv-filer` 接收过滤条件。
- 返回：`future<result>`。

### register-interconnect-recv

根据过滤条件订阅设备端快应用发送来的 Interconnect 消息。注册成功后，插件会在 `on_event` 中收到 `interconnect-message` 事件。

- 参数：
  - `addr: string` 设备地址。
  - `pkg-name: string` 快应用包名。
- 返回：`future<result>`。

### register-deeplink-action

订阅插件 Deeplink 事件。注册后，当浏览器打开 `astrobox://open?source=openPlugin&pluginName=<plugin_name>&data=<plugin_data>` 并拉起 AstroBox 时，插件会在 `on_event` 中收到 `deeplink-action` 事件，`data` 字符串会作为 `event-payload`。只支持订阅一次。

- 返回：`future<result>`。
- 说明：将插件注册为 Deeplink 行为处理方。

### register-provider

注册一个社区源。类型为 `URL` 时，宿主会直接把它当成远程源；类型为 `CUSTOM` 时，宿主会在执行社区源操作时以 `provider-action` 事件回调插件，你需要自行返回数据。支持注册多个 Provider。

- 参数：
  - `name: string` Provider 名称。
  - `provider-type: provider-type` Provider 类型。
- 返回：`future<result>`。

### register-card

注册一个设备详情页卡片。类型为 `TEXT` 时只支持渲染纯文本；类型为 `ELEMENT` 时可以在 `on_card_render` 或 `on_ui_render` 时调用 UI 接口渲染元素卡片。支持注册多个详情页卡片。

- 参数：
  - `card-type: card-type` 卡片类型。
  - `id: string` 卡片标识。
  - `name: string` 卡片展示名称。
- 返回：`future<result>`。

## 权限要求

- `register-transport-recv` 需要声明 `register_transport_recv`
- `register-interconnect-recv` 需要声明 `register_interconnect_recv`
- `register-deeplink-action` 需要声明 `register_deeplink_action`
- `register-provider` 需要声明 `register_provider`
- `register-card` 当前没有额外权限校验

如果权限没声明，或者用户拒绝授权，上面前四个接口都会返回 `Err(())`。

## 注意事项

- 当前这组接口的主要作用是把注册信息写入插件运行时内存里的 `register_state`。
- `register-interconnect-recv` 目前是按“设备地址 + 包名”做精确匹配，不支持通配。
- `register-deeplink-action` 只允许成功注册一次；第二次调用会返回 `Err(())`。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn register_plugin_capabilities(addr: &str) -> Result<(), ()> {
    psys_host::register::register_interconnect_recv(addr, "com.example.remote").await?;
    psys_host::register::register_card(
        psys_host::register::CardType::Element,
        "status-card",
        "状态面板",
    )
    .await?;
    Ok(())
}
```

```go tab="Go"
package plugin

import (
	"errors"
	register "astroboxplugin/bindings/astrobox_psys_host_register"
)

func registerPluginCapabilities(addr string) error {
	ret := register.RegisterInterconnectRecv(addr, "com.example.remote").Read()
	if ret.IsErr() {
		return errors.New("register-interconnect-recv failed")
	}

	ret = register.RegisterCard(
		register.CardTypeElement,
		"status-card",
		"状态面板",
	).Read()
	if ret.IsErr() {
		return errors.New("register-card failed")
	}

	return nil
}
```
