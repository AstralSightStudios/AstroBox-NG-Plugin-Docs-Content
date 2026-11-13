---
title: 插件接管资源安装
---

资源作者可以在资源的 `manifest_v2.json` 里指定一个插件，由它接管资源页的**下载/安装按钮**：
用户一点，AstroBox 不再走常规下载，而是调用这个插件，把资源、目标设备和作者写的数据交给它。

典型用法：

- 资源需要**核销**（兑换码、卡密、授权码）后才给安装包——插件带用户完成核销，再把包入队安装。
- 安装包由作者自己的服务分发，不经过 AstroBox 资源源。
- 安装前要做额外检查或准备。

需要 **Level 4** 插件，并且目标世界实现了 `resource-install` 导出。

## 资源 manifest 怎么写

在 `ext` 里加 `pluginInstall`：

```json
{
  "ext": {
    "pluginInstall": {
      "plugin": "my-plugin",
      "data": { "sku": "abc" },
      "fallback": "block"
    }
  }
}
```

| 字段 | 说明 |
|---|---|
| `plugin` | 必填。插件名，与插件 manifest 的 `name` 一致。必须是合法的插件目录名（不含 `/` `\` `:` 和控制字符，不是 `.` `..`，不以点结尾，最长 128） |
| `data` | 选填。任意 JSON，原样交给插件，序列化后**不超过 16 KiB**（按 UTF-8 字节算）。宿主不解读 |
| `fallback` | 选填，`block`（默认）或 `download`。插件用不了（没装、被停用、没运行起来、客户端过旧）时：`block` 提示用户并停下，`download` 退回常规下载 |

写得不对的声明（插件名不合法、`data` 超限……）**客户端会整体忽略**，等于没写——用户点按钮仍走常规下载。
使用 [AstroBox 创作者控制台](/docs/creator-tools) 发布时，表单里有「插件接管安装」区块：可以直接从插件市场里选插件，写错的声明会在发布前被拦住；填的插件如果不在市场里、或市场里的版本低于 API Level 4，表单会当场提醒，审核时也会核对（会提示并停下的资源，插件不可用会被判为不通过）。

> **资源 manifest 是公开的**，`data` 任何人都看得到，不要把密钥、私有令牌写进去。

接管作用于资源页的下载/安装按钮，**也包括这个资源作为别的资源的绑定资源被一起安装时**：宿主同样先交给插件。
插件返回 `redeem`、或者用户还没装这个插件时，整个绑定安装会停下（别的依赖和主资源都不入队），等用户处理完再点一次；
插件返回 `failed` 或不可用时，绑定安装弹窗会提示原因，用户可以改选后再试。

**批量更新、静默强制更新、合集「一键入队」都不会被接管**：这类资源一律需要用户自己在资源页点按钮
（合集一键入队会跳过它们），宿主不会绕过插件替你下载。

## 插件要实现 handler

目标世界换成 `psys-world-v4-resource-install`，它在基础世界之上多一个导出：

```wit
world psys-world-v4-resource-install {
  include psys-world-v4;
  export astrobox:psys-plugin-v4/resource-install;
}
```

```wit
record install-request {
    request-id: string,        // 本次点击的唯一 id，只用于日志对账
    provider: string,          // 资源所在的源
    resource-id: string,
    resource-name: string,
    resource-type: string,     // quick_app / watchface / firmware / canopus / res_pack / abp / music
    download-key: string,      // 用户当前选中的下载项 key（设备代号）
    version: string,
    version-code: option<u32>,
    device-addr: string,       // 要装到哪台设备
    is-update: bool,           // 是不是在更新已装的版本
    data: string,              // ext.pluginInstall.data 的 JSON 文本，没写则是 "null"
}

variant install-outcome {
    queued,
    redeem(string),
    pass,
    failed(string),
}

on-install: async func(request: install-request) -> install-outcome;
```

### 返回值怎么选

| 返回 | 宿主的行为 |
|---|---|
| `queued` | 插件已经入队（`queue.enqueue-install`）或自己传完了。宿主提示「已开始」，不再走常规下载 |
| `redeem(msg)` | 需要用户先去插件页核销。宿主弹出 `msg`（可留空，留空用默认文案）并**跳转到插件页**。插件在返回前应记下这次请求，页面渲染时好接着上下文走 |
| `pass` | 这次不归插件管，宿主照常走普通下载 |
| `failed(msg)` | 失败，`msg` 是给用户看的原因 |

时限是 **3 分钟**，超时按失败处理——用户在界面上等着，别在里面做长时间的事；需要长时间的传输请入队后返回 `queued`。

### 示例（Rust）

```rust
wit_bindgen::generate!({ path: "wit", world: "psys-world-v4-resource-install", generate_all });

use crate::exports::astrobox::psys_plugin_v4::resource_install::{Guest, InstallOutcome, InstallRequest};

impl Guest for Plugin {
    async fn on_install(request: InstallRequest) -> InstallOutcome {
        // data 来自资源 manifest：是不可信输入，只当数据读，别拿去拼路径、拼命令。
        let Ok(data) = serde_json::from_str::<serde_json::Value>(&request.data) else {
            return InstallOutcome::Failed("资源配置有误".into());
        };
        let Some(sku) = data.get("sku").and_then(|v| v.as_str()) else {
            return InstallOutcome::Pass;
        };

        if !my_store::is_redeemed(sku).await {
            remember_pending(&request); // 记下上下文，插件页渲染时接着走
            return InstallOutcome::Redeem("请先在插件页输入兑换码".into());
        }

        match download_package(sku).await {
            Ok(path) => match queue::enqueue_install(queue::InstallTask {
                name: request.resource_name,
                description: None,
                icon: None,
                device_addr: request.device_addr,
                res_type: queue::ResourceType::Watchface,
                source: queue::InstallSource::File(path),
                watchface_id: None,
            })
            .await
            {
                Ok(_) => InstallOutcome::Queued,
                Err(reason) => InstallOutcome::Failed(reason),
            },
            Err(reason) => InstallOutcome::Failed(reason),
        }
    }
}
```

## 用户没装这个插件会怎样

`fallback` 为 `block` 时，宿主会在插件市场里找这个插件，问用户要不要一键安装；装好后提示用户再点一次按钮。
找不到这个插件、被停用、没运行起来则只提示，不下载。

## 兼容性

这是原生壳的新接口，**旧版客户端没有它**：旧客户端点按钮时会提示「当前版本不支持」
（`fallback` 为 `download` 则直接走常规下载）。作者如果希望旧客户端也能用，请把 `fallback` 设为 `download`，
并确保常规包体本身可用。

## 安全

- 宿主只保证 `data` 是合法 JSON 且不超过 16 KiB，内容一概不解读——它等同于「网上抄来的输入」。
- 资源审核会重点核对：接管的插件是否在插件市场可获取、`data` 内容是否合理、常规下载包体是否合规。
- 插件通过 `queue.enqueue-install` 创建的任务、`mass.send` 发出的数据，各自走自己的权限提示，
  接管安装本身**不会替用户授予任何权限**。
