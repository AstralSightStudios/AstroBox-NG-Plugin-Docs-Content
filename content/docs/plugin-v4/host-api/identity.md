---
title: 宿主标识与账号资料
---

**Level 4 起可用**。两个接口都会弹出授权提示，用户可以随时在插件权限设置里撤销。

## 宿主设备标识

`astrobox:psys-host-v4/os` 的 `device-id`，权限 `os.device-id`。

```wit
device-id: async func() -> result<string, string>;
```

返回形如 `ab-install-<32 位十六进制>` 的字符串，标识的是**运行 AstroBox 的这一份安装**：

- 第一次调用时随机生成，保存在应用本地数据目录，之后重启、升级都不变。
- **不是**硬件序列号，也不从 MAC、IMEI 等派生。卸载重装或清除应用数据后会变成新值。
- 同一台设备上的所有插件拿到的是同一个值，插件之间可以据此关联同一用户，这也是它需要授权的原因。
- 需要识别手环／手表时，用 `device` 接口返回的 `addr`，不要用这个值。

## 已登录账号资料

`astrobox:psys-host-v4/account`，权限 `account.profile`。

```wit
get-current: async func() -> result<option<profile>, string>;
```

- 未登录 AstroBox 账号时返回 `ok(none)`。
- `profile.id` 是 AstroBox 账号 ID，`name`／`username`／`avatar` 为展示资料，`source` 是当前账号源（`casAstralsight` 或 `waterFlames`）。
- `bindings` 固定按 `bandbbs`（米坛社区）、`afdian`（爱发电）、`github` 的顺序各给一条。

每条 `binding`：

| 字段 | 含义 |
|---|---|
| `provider` | `bandbbs` / `afdian` / `github` |
| `status` | `linked`、`unlinked`，或 `unavailable`（本次没能从服务器确认） |
| `id` | 第三方账号 ID，仅 `linked` 时有值 |
| `username` / `display-name` / `avatar` | 展示资料，可能为空 |

```rust
use crate::astrobox::psys_host_v4::account::{self, BindingStatus};

if let Some(profile) = account::get_current().await? {
    for binding in &profile.bindings {
        if binding.provider == "github" && binding.status == BindingStatus::Linked {
            // binding.id 是 GitHub 用户 ID
        }
    }
}
```

### 行为与限制

- **不返回任何凭据**：AstroBox 与第三方的 access token、refresh token、Cookie 都不会出现在结果里。插件需要调用第三方接口时，请让用户在插件里自行授权（例如用[应用内浏览器](./browser)）。
- 绑定状态每次调用都会向服务器实时查询，单项最多等 10 秒。某一项查询失败时该项为 `unavailable`，其余项照常返回；不会用本地缓存去猜测“已绑定”。
- 爱发电和 GitHub 的 `username`、`display-name`、`avatar` 来自登录时同步的资料缓存，只有当缓存里的 ID 与服务器返回的绑定 ID 一致时才会填入，否则只给 `id`。
- 如果查询途中用户切换或退出了账号，调用返回错误，重试即可。
