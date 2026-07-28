---
title: Transport 接口
---

用于与穿戴设备进行底层二进制传输，以及协议数据和 JSON 之间的转换。

## 接口定义

```wit
interface transport {
    enum protocol {
        XIAOMI-VELA-V5-PROTOBUF,
    }

    send: func(device-addr: string, data: list<u8>) -> future;
    request: func(device-addr: string, data: list<u8>)
        -> future<result<list<u8>>>;
    to-json: func(protocol: protocol, data: list<u8>) -> string;
    from-json: func(protocol: protocol, data: string) -> result<list<u8>>;
}
```

## 类型

### protocol

- `XIAOMI-VELA-V5-PROTOBUF`：基于 Xiaomi Vela 5 的 Protobuf 传输协议

## 函数

### send

- 参数：
  - `device-addr: string` 设备地址。
  - `data: list<u8>` 待发送的二进制数据。
- 返回：`future<()>`。
- 说明：仅发送数据，不等待响应。

### request

- 参数：
  - `device-addr: string` 设备地址。
  - `data: list<u8>` 待发送的二进制数据。
- 返回：`future<result<list<u8>>>`，成功返回响应数据。
- 说明：请求-响应模式。

### to-json

- 参数：
  - `protocol: protocol` 协议类型。
  - `data: list<u8>` 二进制数据。
- 返回：`string`。
- 说明：将协议数据转换为 JSON 字符串，便于日志与调试。

### from-json

- 参数：
  - `protocol: protocol` 协议类型。
  - `data: string` JSON 字符串。
- 返回：`result<list<u8>>`。
- 说明：将 JSON 反序列化为协议二进制数据。

## 当前状态

> `transport` 接口已经出现在 WIT 中，但当前宿主实现仍接近占位状态。
>
> `send` / `request` 目前不要当作稳定可用的数据通道；
> `to-json` / `from-json` 也还没有可依赖的协议转换实现。
>
> 如果你要和设备上的快应用做实际应用层通信，当前应优先使用 [Interconnect](./interconnect)。

## 注意事项

暂无

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn request_packet(addr: &str, data: Vec<u8>) -> Result<Option<Vec<u8>>, ()> {
    psys_host::transport::send(addr, &data).await;
    match psys_host::transport::request(addr, &data).await {
        Ok(resp) => Ok(Some(resp)),
        Err(()) => Ok(None),
    }
}
```

```go tab="Go"
package plugin

import (
	transport "astroboxplugin/bindings/astrobox_psys_host_transport"
)

func requestPacket(addr string, data []byte) ([]byte, bool) {
	_ = transport.ToJson(transport.ProtocolXiaomiVelaV5Protobuf, data)

	ret := transport.Request(addr, data).Read()
	if ret.IsErr() {
		return nil, false
	}
	return ret.Ok(), true
}
```
