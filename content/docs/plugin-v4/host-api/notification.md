---
title: 设备通知与实时活动
---

`astrobox:psys-host-v4/notification`，**Level 4 起可用**。

向连接的穿戴设备发送通知，或通过小米协议的 `NotifyData.focus` / `focus_v2`
发送和更新实时活动（超级岛）。这不是手机／电脑本机的系统通知，也不是 iOS ActivityKit。

## 权限

在 manifest 的 `permissions` 中加入 `notification`。首次调用会请求用户授权；
用户可以在插件权限设置中撤销授权。发送和撤回都会检查权限。

## 接口

```wit
send: async func(addr: string, message: message) -> result<_, string>;
remove: async func(addr: string, id: u32) -> result<_, string>;
```

`message` 字段：

| 字段 | 类型 | 含义 |
|---|---|---|
| `id` | `u32` | 插件自己分配的通知 ID；更新和撤回使用同一个 ID |
| `app-name` | `string` | 通知来源显示名 |
| `title` | `string` | 标题 |
| `sub-title` | `string` | 副标题，无则空串 |
| `body` | `string` | 正文 |
| `timestamp-ms` | `option<u64>` | Unix 毫秒时间戳；省略则用宿主当前时间 |
| `live-activity` | `option<live-activity>` | 省略则是普通通知；否则选择 `focus` 或 `focus-v2` |

普通通知的 JavaScript 示例：

```js
import { send, remove } from 'astrobox:psys-host-v4/notification';

await send(deviceAddr, {
  id: 1,
  appName: '任务提醒',
  title: '任务已完成',
  subTitle: '',
  body: '可以查看结果了',
  timestampMs: undefined,
  liveActivity: undefined,
});

// 撤回本插件刚才发出的通知。
await remove(deviceAddr, 1);
```

## 实时活动

`live-activity` 是 variant，JS 中用 `{ tag: 'focus', val: {...} }` 或
`{ tag: 'focus-v2', val: {...} }` 表示。

- `focus` 包含 `style`、`title`、`content`、`desc`、可选 `progress`、`updatable`、`sequence`。
- `focus-v2` 包含 `scene`、`ticker`、`basic-info`、可选 `hint-info` 和 `progress`、`updatable`、`sequence`。
- `text` 为 `{ chars: string, color: option<list<u8>> }`。
- `progress` 为 `{ section-count: u32, progress: u32, color: option<list<u8>> }`。
- `info` 包含必填的 `title: text`，及可选的 `sub-title`、`content`、`sub-content`、
  `special-title`（均为 `text`）和 `special-title-bg: list<u8>`。

更新时保持 `id` 不变，设置 `updatable: true` 并递增 `sequence`，再次调用 `send`。
结束时使用 `remove`。同一个消息不能同时携带两版实时活动。

`style`、`scene`、进度数值和颜色字节是协议原值，当前 proto 没有提供可跨型号保证有效的
枚举或颜色编码说明。请使用目标设备已经确认支持的取值，不要把任意数字视为通用样式。

## 行为与限制

- 当前实现的是**小米通知协议**。其他设备协议会明确返回不支持，不会伪报发送成功。
- 设备必须存在于当前连接的 ECS 实例中并已认证。
- 宿主由插件身份及 `id` 生成协议通知标识，插件不能指定其他应用的包名、通知 key 或 uid。
  同一插件重新加载后仍可用同一个 `id` 更新／撤回；换插件名会改变命名空间。
- `send` 也用于更新普通通知。最终更新／显示行为由设备固件决定。
- 返回成功只表示**已加入传输队列**；此接口不等待设备显示确认。
- 单个序列化通知包最多 16 KiB。
- 当前不提供通知点击或快捷回复回调，所以 `support-open` 和 `support-reply` 固定关闭。
- 实时活动的显示效果、样式和固件兼容性仍需真机验证。不要据此推断所有小米设备均支持超级岛。
