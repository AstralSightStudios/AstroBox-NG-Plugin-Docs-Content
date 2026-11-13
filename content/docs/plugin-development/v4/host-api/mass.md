---
title: MASS 数据传输
---

`astrobox:psys-host-v4/mass`，**Level 4 起可用**。

把一块数据当作 MASS 载荷直接发给小米穿戴设备，并等设备确认安装结果。表盘、快应用、
固件的安装本质上都走这条通道——这个接口把它开放给插件，用来做「自己有一套分发渠道、
自己拿到安装包」的场景。

> 需要排队、进度条、断线重试、压缩包处理，请改用 [`queue.enqueue-install`](./queue)。
> `mass.send` 是一条没有任何包装的直通管道。

## 权限

| 权限 | 作用 |
|----|----|
| `mass` | 向设备发送表盘、快应用、通知图标。首次调用会请求用户授权，授权可在插件权限设置里撤销 |
| `mass.firmware` | 发送固件。**每次调用都会弹窗确认，授权不会被记住** |

固件会改写设备系统，宿主因此把它单列出来，且无法被「始终允许」。弹窗会先让用户等待几秒才能确认。

## 接口

```wit
enum data-type { watchface, quickapp, firmware, notification-icon }

record send-options {
    watchface-id: option<string>,
    package-name: option<string>,
    progress-event: option<string>,
}

send: async func(device-addr: string, data-type: data-type,
                 data: list<u8>, options: send-options) -> result<_, string>;
```

| 字段 | 说明 |
|---|---|
| `device-addr` | 已连接设备的地址，见 `device.get-connected-device-list` |
| `data` | **原始载荷**，宿主不解包、不改写：表盘是设备认的 `.bin`，快应用是 `.rpk`，固件是 OTA 文件 |
| `watchface-id` | 仅表盘：改写表盘 ID |
| `package-name` | 快应用 / 通知图标的包名。快应用不填则用占位包名 `a.b.c`；**通知图标必须填**。只允许字母、数字、`.` `_` `-`，最长 128 |
| `progress-event` | 想收传输进度就填一个事件名，见下 |

`.mwz` 之类的压缩包要先自己解开再发。想让宿主代劳，用 `queue.enqueue-install`。

## 进度

`progress-event` 填了的话，传输期间宿主会以**插件消息**（`on-event` 的 `plugin-message`）送回：

```json
{ "eventName": "<你填的事件名>", "payload": "{\"progress\": 0.42}" }
```

`progress` 在 0 到 1 之间，不会比每 250 ms 更频繁，成功结束时一定有一条 `1.0`。

## JavaScript 示例

```js
import { send } from 'astrobox:psys-host-v4/mass';

const result = await send(deviceAddr, 'watchface', binBytes, {
  watchfaceId: undefined,
  packageName: undefined,
  progressEvent: 'my-plugin.progress',
});
```

失败时返回 `Err(reason)`（JS 中表现为抛出异常），`reason` 是给人看的英文原因，不要拿它做逻辑分支。

## 行为与限制

- 仅支持**小米协议**设备，其他设备会明确返回不支持。
- 同一台设备同一时间只能有一个安装在进行，撞上正在进行的安装会直接返回错误；
  `queue.enqueue-install` 创建的任务和用户自己加的任务都算「正在进行的安装」。
- 单次载荷上限 1 GiB；插件自身有 256 MiB 内存上限，`data` 走不到这么大。
  更大的包请走 `queue.enqueue-install` 的 `file` 来源，它不占插件内存。
- 发送前宿主会把数据落到自己的暂存目录，结束后删除，插件不需要也不应该关心路径。
- **宿主不校验载荷内容**。发了错误的 `.bin` 的后果由插件和设备固件承担，建议自己先校验。
