---
title: Watchface 接口 (API Level 3+)
---

用于读取设备上的表盘列表，并切换当前表盘。

## 接口定义

```wit
interface watchface {
    record watchface-info {
        id: string,
        name: string,
        is-current: bool,
    }

    get-watchface-list: func(addr: string) -> future<result<list<watchface-info>>>;
    set-current-watchface: func(addr: string, watchface-id: string) -> future<result>;
}
```

## 类型

### watchface-info

- `id`：表盘 ID。
- `name`：表盘名称。
- `is-current`：是否为当前表盘。

## 函数

### get-watchface-list

- 参数：`addr: string` 设备地址。
- 返回：`future<result<list<watchface-info>>>`。

### set-current-watchface

- 参数：
  - `addr: string` 设备地址。
  - `watchface-id: string` 目标表盘 ID。
- 返回：`future<result>`。

## 权限要求

- 这组接口都会触发 `watchface` 权限校验。
- 如果插件未声明 `watchface`，或用户拒绝授权，会返回 `Err(())`。

## 注意事项

- `get-watchface-list` 当前会通过设备侧 `ResourceSystem` 主动请求表盘列表，并等待设备回包。
- 如果设备不存在、资源系统不可用、或者设备未返回表盘列表，会返回 `Err(())`。
- `set-current-watchface` 当前会先检查 `watchface-id` 非空；空字符串会直接返回 `Err(())`。
- 真正切换表盘时，宿主当前调用的是设备侧 `WatchfaceSystem::set_watchface`。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn switch_to_first_non_current(addr: &str) -> Result<(), ()> {
    let list = psys_host::watchface::get_watchface_list(addr).await?;
    let Some(target) = list.into_iter().find(|item| !item.is_current) else {
        return Ok(());
    };

    psys_host::watchface::set_current_watchface(addr, &target.id).await
}
```

```go tab="Go"
package plugin

import (
	"errors"
	watchface "astroboxplugin/bindings/astrobox_psys_host_watchface"
)

func switchToFirstNonCurrent(addr string) error {
	ret := watchface.GetWatchfaceList(addr).Read()
	if ret.IsErr() {
		return errors.New("get-watchface-list failed")
	}

	for _, item := range ret.Ok() {
		if item.IsCurrent {
			continue
		}
		setRet := watchface.SetCurrentWatchface(addr, item.Id).Read()
		if setRet.IsErr() {
			return errors.New("set-current-watchface failed")
		}
		return nil
	}

	return nil
}
```
