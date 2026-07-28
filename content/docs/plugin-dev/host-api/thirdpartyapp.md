---
title: ThirdpartyApp 接口
---

管理和启动设备上的第三方快应用。

## 接口定义

```wit
interface thirdpartyapp {
    record app-info {
        package-name: string,
        fingerprint: list<u32>,
        version-code: u32,
        can-remove: bool,
        app-name: string
    }

    launch-qa: func(addr: string, app-info: app-info, page-name: string) -> future<result>;
    get-thirdparty-app-list: func(addr: string) -> future<result<list<app-info>>>;
}
```

## 类型

### app-info

- `package-name`：包名。
- `fingerprint`：签名指纹。
- `version-code`：版本号。
- `can-remove`：是否允许卸载。
- `app-name`：应用名称。

## 函数

### launch-qa

- 参数：
  - `addr: string` 设备地址。
  - `app-info: app-info` 目标应用信息。
  - `page-name: string` 目标页面。
- 返回：`future<result>`。

### get-thirdparty-app-list

- 参数：`addr: string` 设备地址。
- 返回：`future<result<list<app-info>>>`。
- 说明：成功时返回应用列表；失败时返回 `Err(())`。

## 权限要求

- 这组接口都会触发 `thirdpartyapp` 权限校验。
- 如果插件未声明 `thirdpartyapp`，或用户拒绝授权，会返回 `Err(())`。

## 注意事项

- `get-thirdparty-app-list` 当前会通过设备侧 `ResourceSystem` 主动请求快应用列表，并等待设备回包。
- 如果设备不存在、资源系统不可用、或者设备没有按预期回包，会返回 `Err(())`。
- `launch-qa` 会优先使用你传入的 `app-info`；如果 `package-name` 有值但 `fingerprint` 为空，宿主会尝试从当前设备已安装应用里按包名补全签名信息。
- 如果 `package-name` 为空，或者目标设备上找不到对应应用，`launch-qa` 会返回 `Err(())`。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host;

pub async fn launch_first_app(addr: &str) -> Result<(), ()> {
    let ret = psys_host::thirdpartyapp::get_thirdparty_app_list(addr).await;
    let apps = ret?;
    let Some(app) = apps.into_iter().next() else {
        return Ok(());
    };

    psys_host::thirdpartyapp::launch_qa(addr, &app, "pages/index").await
}
```

```go tab="Go"
package plugin

import (
	"errors"
	thirdpartyapp "astroboxplugin/bindings/astrobox_psys_host_thirdpartyapp"
)

func launchFirstApp(addr string) error {
	ret := thirdpartyapp.GetThirdpartyAppList(addr).Read()
	if ret.IsErr() {
		return errors.New("get-thirdparty-app-list failed")
	}

	apps := ret.Ok()
	if len(apps) == 0 {
		return nil
	}

	launchRet := thirdpartyapp.LaunchQa(addr, apps[0], "pages/index").Read()
	if launchRet.IsErr() {
		return errors.New("launch-qa failed")
	}
	return nil
}
```
