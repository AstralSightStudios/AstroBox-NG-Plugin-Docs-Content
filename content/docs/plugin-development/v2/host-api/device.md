---
title: Device 接口
---

用于获取宿主当前识别到的设备列表，并主动断开指定设备。

## 接口定义

```wit
interface device {
    record device-info {
        name: string,
        addr: string
    }

    get-device-list: func() -> future<list<device-info>>;
    get-connected-device-list: func() -> future<list<device-info>>;
    disconnect-device: func(addr: string) -> future<result>;
}
```

## 类型

### device-info

- `name`：设备名称。
- `addr`：设备地址。后续调用 `transport`、`interconnect`、`thirdpartyapp`、`watchface` 等接口时都会用到它。

## 函数

### get-device-list

- 返回：`future<list<device-info>>`。
- 说明：返回宿主当前可枚举到的全部设备，不要求设备已经连接。

### get-connected-device-list

- 返回：`future<list<device-info>>`。
- 说明：只返回当前已连接设备。

### disconnect-device

- 参数：`addr: string` 设备地址。
- 返回：`future<result>`。
- 说明：成功时返回 `Ok(())`，失败时返回 `Err(())`。

## 权限要求

- 这组接口都会触发 `device` 权限校验。
- 如果插件 `manifest.json` 没声明 `device`，或者用户拒绝授权：
  - `get-device-list` / `get-connected-device-list` 会返回空列表。
  - `disconnect-device` 会返回 `Err(())`。

## 注意事项

- `get-device-list` 当前读取的是宿主前端保存的设备记录历史，不是一次实时蓝牙扫描。
- `get-connected-device-list` 读取的是宿主运行时里当前仍然在线的 `XiaomiDevice` 实例。
- `disconnect-device` 当前通过主窗口执行前端 `miwear_disconnect` 调用；如果主窗口不存在，或者前端脚本执行失败，会返回 `Err(())`。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn disconnect_all_connected() {
    let devices = psys_host::device::get_connected_device_list().await;

    for device in devices {
        match psys_host::device::disconnect_device(&device.addr).await {
            Ok(()) => tracing::info!("disconnected {}", device.addr),
            Err(()) => tracing::warn!("disconnect failed: {}", device.addr),
        }
    }
}
```

```go tab="Go"
package plugin

import (
	device "astroboxplugin/bindings/astrobox_psys_host_device"
	"fmt"
)

func listConnectedDevices() {
	devices := device.GetConnectedDeviceList().Read()
	for _, d := range devices {
		fmt.Printf("%s (%s)\n", d.Name, d.Addr)
	}
}
```
