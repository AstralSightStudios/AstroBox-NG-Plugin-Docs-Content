---
title: Dialog 接口
---

用于在宿主 UI 中弹出对话框、选择本地文件、发起保存会话，以及打开外部 URL。

## 接口定义

```wit
interface dialog {
    enum dialog-type {
        ALERT,
        INPUT
    }

    enum dialog-style {
        WEBSITE,
        SYSTEM
    }

    record dialog-button {
        id: string,
        primary: bool,
        content: string
    }

    record dialog-info {
        title: string,
        content: string,
        buttons: list<dialog-button>
    }

    record dialog-result {
        clicked-btn-id: string,
        input-result: string
    }

    record pick-config {
        read: bool,
        copy-to: option<string>,
    }

    record filter-config {
        multiple: bool,
        extensions: list<string>,
        default-directory: string,
        default-file-name: string,
    }

    record pick-result {
        name: string,
        data: list<u8>,
    }

    record save-session {
        session-id: u64,
        name: string,
    }

    show-dialog: func(dialog-type: dialog-type, style: dialog-style, info: dialog-info) -> future<dialog-result>;

    pick-file: func(config: pick-config, filter: filter-config) -> future<pick-result>;

    save-file-start: func(filter: filter-config) -> future<result<save-session>>;
    save-file-write-chunk: func(session-id: u64, data: list<u8>) -> future<result>;
    save-file-finish: func(session-id: u64) -> future<result>;
    save-file-abort: func(session-id: u64) -> future;

    open-url: func(url: string);
}
```

## 类型

### dialog-type

- `ALERT`：提示/确认框。
- `INPUT`：带输入框的对话框。

### dialog-style

- `WEBSITE`：Web 风格弹窗。
- `SYSTEM`：系统原生弹窗。

### dialog-button

- `id`：按钮标识，点击后会回传到 `dialog-result.clicked-btn-id`。
- `primary`：是否为主按钮。
- `content`：按钮文字。

### dialog-info

- `title`：标题。
- `content`：正文。
- `buttons`：按钮列表。

### dialog-result

- `clicked-btn-id`：用户点击的按钮 ID。
- `input-result`：输入框内容。只有 `INPUT` 类型才会用到。

### pick-config

- `read`：是否让宿主读取文件内容并返回到 `pick-result.data`。
- `copy-to`：可选复制目标路径。它是插件根目录下的相对路径，不允许绝对路径和 `..`。

### filter-config

- `multiple`：是否允许多选。
- `extensions`：扩展名过滤。
- `default-directory`：默认打开目录。
- `default-file-name`：默认文件名。

### pick-result

- `name`：文件名。
- `data`：文件内容字节。只有 `pick-config.read = true` 时才会返回。

### save-session

- `session-id`：保存会话 ID。
- `name`：宿主最终选中的文件名。

## 函数

### show-dialog

- 参数：
  - `dialog-type: dialog-type`
  - `style: dialog-style`
  - `info: dialog-info`
- 返回：`future<dialog-result>`。

### pick-file

- 参数：
  - `config: pick-config`
  - `filter: filter-config`
- 返回：`future<pick-result>`。

### save-file-start

- 参数：`filter: filter-config`
- 返回：`future<result<save-session>>`。
- 说明：成功时返回保存会话；失败或用户取消时返回 `Err(())`。

### save-file-write-chunk

- 参数：
  - `session-id: u64`
  - `data: list<u8>`
- 返回：`future<result>`。

### save-file-finish

- 参数：`session-id: u64`
- 返回：`future<result>`。

### save-file-abort

- 参数：`session-id: u64`
- 返回：`future<()>`。

### open-url

- 参数：`url: string`
- 返回：`()`。
- 说明：同步接口，通常会调用宿主系统浏览器打开 URL。

## 当前行为

- `show-dialog` 的 `ALERT + SYSTEM` 组合会走系统原生弹窗。
- `show-dialog` 的 `WEBSITE` 风格支持 `ALERT` 和 `INPUT`。
- `INPUT + SYSTEM` 当前没有宿主实现，调用后通常会拿到默认空结果。
- 系统弹窗最多只支持 3 个按钮，超出的按钮会被宿主截断。
- 系统弹窗会优先把 `primary = true` 的按钮排到前面，再映射到系统按钮顺序。
- `pick-file` 虽然 `filter-config.multiple` 存在，但当前 WIT 返回值仍然只有一个 `pick-result`；如果多选，宿主目前只会返回第一项。
- `pick-file` 在用户取消时不会返回 `Err`，而是返回空的 `pick-result`：`name = ""`，`data = []`。
- 如果 `copy-to` 有值但 `read = false`，宿主仍会先读取文件内容完成复制，只是不会把字节返回给插件。
- `copy-to` 最终会被拼接到插件根目录下；绝对路径和包含 `..` 的路径会被拒绝。
- `save-file-*` 是分块保存链路：`save-file-start -> save-file-write-chunk -> save-file-finish`。如果中途取消，调用 `save-file-abort`。
- `save-file-start` 在用户取消时返回 `Err(())`，这点和 `pick-file` 不同。
- 保存会话按 `(plugin-name, session-id)` 隔离，不同插件之间不会共用会话。
- `open-url` 是 fire-and-forget；如果宿主打开浏览器失败，不会把错误回传给插件。

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host::dialog::{
    self, DialogButton, DialogInfo, DialogStyle, DialogType, FilterConfig,
};

const CONFIRM_ID: &str = "confirm";
const CANCEL_ID: &str = "cancel";

pub async fn prompt_name() -> Option<String> {
    let ret = dialog::show_dialog(
        DialogType::Input,
        DialogStyle::Website,
        &DialogInfo {
            title: "输入名称".into(),
            content: "请输入新的配置名称".into(),
            buttons: vec![
                DialogButton {
                    id: CONFIRM_ID.into(),
                    primary: true,
                    content: "确认".into(),
                },
                DialogButton {
                    id: CANCEL_ID.into(),
                    primary: false,
                    content: "取消".into(),
                },
            ],
        },
    )
    .await;

    if ret.clicked_btn_id != CONFIRM_ID {
        return None;
    }

    let value = ret.input_result.trim().to_string();
    if value.is_empty() {
        None
    } else {
        Some(value)
    }
}

pub async fn export_text(default_file_name: &str, text: &str) -> Result<(), ()> {
    let session = dialog::save_file_start(&FilterConfig {
        multiple: false,
        extensions: vec!["txt".into()],
        default_directory: "".into(),
        default_file_name: default_file_name.into(),
    })
    .await?;

    dialog::save_file_write_chunk(session.session_id, text.as_bytes()).await?;
    dialog::save_file_finish(session.session_id).await
}
```

```go tab="Go"
package plugin

import (
	dialog "astroboxplugin/bindings/astrobox_psys_host_dialog"
	"fmt"
	"strings"

	"github.com/bytecodealliance/wit-bindgen/wit_types"
)

const (
	dialogConfirmID = "confirm"
	dialogCancelID  = "cancel"
)

func confirmDialog(title string, content string) bool {
	ret := dialog.ShowDialog(
		dialog.DialogTypeAlert,
		dialog.DialogStyleSystem,
		dialog.DialogInfo{
			Title:   title,
			Content: content,
			Buttons: []dialog.DialogButton{
				{Id: dialogConfirmID, Primary: true, Content: "确认"},
				{Id: dialogCancelID, Primary: false, Content: "取消"},
			},
		},
	).Read()
	return ret.ClickedBtnId == dialogConfirmID
}

func pickLocalFile() (string, []byte, error) {
	ret := dialog.PickFile(
		dialog.PickConfig{Read: true, CopyTo: wit_types.None[string]()},
		dialog.FilterConfig{
			Multiple:         false,
			Extensions:       []string{"json", "txt"},
			DefaultDirectory: "",
			DefaultFileName:  "",
		},
	).Read()
	if strings.TrimSpace(ret.Name) == "" {
		return "", nil, fmt.Errorf("未选择文件")
	}
	return ret.Name, ret.Data, nil
}

func saveText(defaultFileName string, content string) error {
	ret := dialog.SaveFileStart(
		dialog.FilterConfig{
			Multiple:         false,
			Extensions:       []string{"txt"},
			DefaultDirectory: "",
			DefaultFileName:  defaultFileName,
		},
	).Read()
	if ret.IsErr() {
		return fmt.Errorf("未选择保存位置")
	}

	session := ret.Ok()
	writeRet := dialog.SaveFileWriteChunk(session.SessionId, []byte(content)).Read()
	if writeRet.IsErr() {
		dialog.SaveFileAbort(session.SessionId).Read()
		return fmt.Errorf("写入本地分块失败")
	}

	finishRet := dialog.SaveFileFinish(session.SessionId).Read()
	if finishRet.IsErr() {
		return fmt.Errorf("完成本地保存失败")
	}
	return nil
}
```
