---
title: Queue 接口
---

向 AstroBox 资源队列添加任务。

## 接口定义

```wit
interface queue {
    enum resource-type {
        quickapp,
        watchface,
        firmware
    }

    add-resource-to-queue: func(res-type: resource-type, file-path: string);
}
```

## 类型

### resource-type

- `quickapp`：快应用资源。
- `watchface`：表盘资源。
- `firmware`：固件资源。

## 函数

### add-resource-to-queue

- 参数：
  - `res-type: resource-type` 资源类型。
  - `file-path: string` 本地文件路径。
- 返回：`()`。
- 说明：这是同步接口，只负责把本地文件路径加入 AstroBox 资源队列；后续安装/刷写流程由宿主管理。

## 权限要求

- 这个接口会触发 `queue` 权限校验。
- 如果插件未声明 `queue`，或者用户拒绝授权，宿主会直接忽略这次调用。

## 注意事项

暂无

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub fn add_firmware(path: &str) {
    psys_host::queue::add_resource_to_queue(
        psys_host::queue::ResourceType::Firmware,
        path,
    );
}
```

```go tab="Go"
package plugin

import queue "astroboxplugin/bindings/astrobox_psys_host_queue"

func enqueueWatchface(path string) {
	queue.AddResourceToQueue(queue.ResourceTypeWatchface, path)
}
```
