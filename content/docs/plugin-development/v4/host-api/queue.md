---
title: 安装队列
---

`astrobox:psys-host-v4/queue`。`add-resource-to-queue` 是老接口；`enqueue-install` 与
`cancel-task` 是 **Level 4 新增**，让插件自己创建、取消安装任务。

## 权限

`queue`。`enqueue-install` 每次都会请求用户授权（授权规则与持久化方式同其它权限），`cancel-task` 只要求声明过
`queue`——它只能动**本插件自己**创建的任务。

## 为什么用队列而不是 `mass.send`

插件创建的任务就是一个普通的队列任务：排队、进度、暂停、重试、断线恢复都和用户自己添加的任务一样，
用户在队列页能看到它、也能自己取消。宿主还会替你做压缩包处理（如 `.mwz`）和安装前检查。
`mass.send` 则是一条不经过这些的直通管道，见 [MASS 数据传输](./mass)。

## 接口

```wit
variant install-source {
    file(string),       // 插件目录内的相对路径
    bytes(list<u8>),    // 直接给内容
}

record install-task {
    name: string,                   // 队列里显示的名字
    description: option<string>,    // 副标题；留空按资源类型给默认说明
    icon: option<string>,           // https URL 或 data:image/ URL
    device-addr: string,
    res-type: resource-type,        // quickapp / watchface / firmware
    source: install-source,
    watchface-id: option<string>,   // 仅表盘：改写表盘 ID
}

enqueue-install: async func(task: install-task) -> result<string, string>;
cancel-task: async func(task-id: string) -> result<_, string>;
```

`enqueue-install` 返回任务 id。**成功只表示已入队，不表示已装上。**
是否自动开始跟随用户的「自动安装本地资源」设置。

### 安装包来源

两种来源都会被**宿主接管**，入队后不要再读写它：

- `file`：插件目录内的相对路径，不能越出插件目录（绝对路径、`..`、指向目录外的符号链接都会被拒绝）。
  宿主把文件**移走**，不再复制一份，所以几百 MB 的包也不占插件内存；任务结束后由宿主删除。
- `bytes`：直接给内容，宿主写进自己的暂存目录，任务结束后删除。受插件 256 MiB 内存上限约束，大包请用 `file`。

单个安装包上限 1 GiB。

### 字段限制

宿主会清洗这些会原样显示在队列里的文字，免得一个插件把队列刷花：

| 字段 | 限制 |
|---|---|
| `name` | 必填，去掉控制字符后最多 120 个字符 |
| `description` | 最多 240 个字符 |
| `icon` | `https://` URL（≤ 2048 字符，不含空白）或 `data:image/…`（≤ 256 KiB），其它一律拒绝 |
| `watchface-id` | 仅表盘；1–16 位字母或数字 |
| `device-addr` | 必填；设备必须已连接，否则返回错误 |

## 任务状态回报

任务状态变化时，宿主向**创建它的插件**发一条插件消息（`on-event` 的 `plugin-message`），
事件名是 `queue.task`：

```json
{ "eventName": "queue.task",
  "payload": "{\"taskId\":\"plugin-…\",\"status\":\"running\",\"progress\":0.42}" }
```

| 字段 | 含义 |
|---|---|
| `taskId` | `enqueue-install` 返回的 id |
| `status` | `pending` / `running` / `waiting_device` / `succeeded` / `failed` / `canceled` |
| `progress` | 0 到 1 |
| `errorCode` / `errorDetail` | 仅失败时带 |

状态变化立即发送，**仅进度变化**时每秒最多一条，且变化不足 1% 不发。插件已停止时消息会被丢弃，
不会影响队列本身。

## 取消

`cancel-task(task-id)`：排队中的直接移除，正在传输的发起取消。只能取消本插件创建的任务，
任务不存在或不是自己的返回错误 `no such task`。

## JavaScript 示例

```js
import { enqueueInstall } from 'astrobox:psys-host-v4/queue';

const taskId = await enqueueInstall({
  name: '某某表盘',
  description: '来自 my-plugin',
  icon: undefined,
  deviceAddr,
  resType: 'watchface',
  source: { tag: 'file', val: 'downloads/face.bin' },
  watchfaceId: undefined,
});
```

## 与资源 manifest 联动

资源作者可以让某个插件接管下载按钮，插件在 [`resource-install`](../resource-install) 的
`on-install` 里调用 `enqueue-install`，就能把「点击下载」变成一个队列任务。
