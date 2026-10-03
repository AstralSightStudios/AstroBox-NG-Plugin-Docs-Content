---
title: OS 接口
---

提供宿主系统的基础信息查询能力，全部为异步调用。

## 接口定义

```wit
interface os {
    arch: func() -> future<string>;
    hostname: func() -> future<string>;
    locale: func() -> future<string>;
    platform: func() -> future<string>;
    version: func() -> future<string>;
    astrobox-language: func() -> future<string>;
    appearance: func() -> future<string>;
    timezone-offset-minutes: func() -> future<s32>;
}
```

## 函数

### arch

- 返回：`future<string>`，CPU 架构字符串。

### hostname

- 返回：`future<string>`，宿主设备名称。

### locale

- 返回：`future<string>`，系统 locale 字符串。

### platform

- 返回：`future<string>`，宿主平台标识。

### version

- 返回：`future<string>`，宿主系统版本号。

### astrobox-language

- 返回：`future<string>`，AstroBox 语言设置。

### appearance

- 返回：`future<string>`，宿主当前外观模式。
- 说明：通常会是类似 `light` / `dark` 的值，具体取值以宿主实现为准。

### timezone-offset-minutes

- 返回：`future<s32>`，宿主当前时区相对 UTC 的分钟偏移量。

## 注意事项

- `arch` 直接来自宿主进程的 `std::env::consts::ARCH`。
- `hostname` 使用 `whoami::fallible::hostname()`；失败时回退为 `"unknown-host"`。
- `locale` 使用系统 locale；拿不到时回退为 `"en-US"`。
- `platform` / `version` 来自 `os_info`。
- `astrobox-language` 和 `appearance` 当前都通过宿主前端查询。
- 这组接口当前没有额外权限校验。

## Rust 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn print_os_info() {
    let arch = psys_host::os::arch().await;
    let platform = psys_host::os::platform().await;
    let version = psys_host::os::version().await;
    let locale = psys_host::os::locale().await;
    let hostname = psys_host::os::hostname().await;
    let language = psys_host::os::astrobox_language().await;
    let appearance = psys_host::os::appearance().await;
    let timezone_offset_minutes = psys_host::os::timezone_offset_minutes().await;

    tracing::info!("arch={}", arch);
    tracing::info!("platform={}", platform);
    tracing::info!("version={}", version);
    tracing::info!("locale={}", locale);
    tracing::info!("hostname={}", hostname);
    tracing::info!("astrobox_language={}", language);
    tracing::info!("appearance={}", appearance);
    tracing::info!("timezone_offset_minutes={}", timezone_offset_minutes);
}
```

```go tab="Go"
package plugin

import (
	"fmt"
	oshost "astroboxplugin/bindings/astrobox_psys_host_os"
)

func printOSInfo() {
	fmt.Println("arch =", oshost.Arch().Read())
	fmt.Println("platform =", oshost.Platform().Read())
	fmt.Println("version =", oshost.Version().Read())
	fmt.Println("locale =", oshost.Locale().Read())
	fmt.Println("hostname =", oshost.Hostname().Read())
	fmt.Println("astroboxLanguage =", oshost.AstroboxLanguage().Read())
	fmt.Println("appearance =", oshost.Appearance().Read())
	fmt.Println("timezoneOffsetMinutes =", oshost.TimezoneOffsetMinutes().Read())
}
```
