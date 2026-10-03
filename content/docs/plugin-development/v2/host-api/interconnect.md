---
title: Interconnect 接口
---

与穿戴设备上的快应用进行字符串消息通信。

## 接口定义

```wit
interface interconnect {
    send-qaic-message: func(device-addr: string, pkg-name: string, data: string) -> future<result>;
}
```

## 函数

### send-qaic-message

- 参数：
  - `device-addr: string` 设备地址。
  - `pkg-name: string` 快应用包名。
  - `data: string` 发送数据字符串，通常是 JSON 或 JSON-RPC 文本。
- 返回：`future<result>`。
- 说明：如果你还需要接收快应用的回包或主动消息，通常要先调用 `register::register-interconnect-recv`，然后在插件事件里处理 `interconnect-message`。

## 权限要求

- 这个接口会触发 `interconnect` 权限校验。
- 如果插件未声明 `interconnect`，或用户拒绝授权，会返回 `Err(())`。

## 注意事项

- 宿主当前会把 `data: string` 转成 UTF-8 字节后发送给快应用。这是通信数据包定义要求，但实际上在设备端收到的还是字符串。
- 发送前会先在设备资源组件中按 `pkg-name` 精确匹配目标快应用。
- 如果设备不存在、快应用不存在、或者发送底层消息失败，都会返回 `Err(())`。
- 当你通过 `register-interconnect-recv` 注册接收后，当前宿主会按“设备地址 + 包名”做精确匹配，再把收到的字符串原样作为 `interconnect-message` 的 `event-payload` 派发给插件。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn send_rpc_ping(addr: &str, pkg_name: &str) -> Result<(), ()> {
    psys_host::interconnect::send_qaic_message(
        addr,
        pkg_name,
        r#"{"id":"req_1","method":"hello"}"#,
    )
    .await
}
```

```go tab="Go"
package plugin

import (
	"errors"
	interconnect "astroboxplugin/bindings/astrobox_psys_host_interconnect"
)

func sendRpcRequest(addr string, pkgName string, payload string) error {
	ret := interconnect.SendQaicMessage(addr, pkgName, payload).Read()
	if ret.IsErr() {
		return errors.New("send-qaic-message failed")
	}
	return nil
}
```
