---
title: UI 接口
---

`ui` 是**旧版** UI Builder 接口。除非你在维护历史插件，否则优先使用 [UI V3](./ui-v3)。

## 接口定义

```wit
interface ui {
    enum element-type {
        BUTTON,
        INPUT,
        SELECT,
        OPTION,
        IMAGE,
        VIDEO,
        AUDIO,
        SVG,
        DIV,
        SPAN,
        P,
    }

    enum flex-direction {
        ROW,
        COLUMN,
        ROW-REVERSE,
        COLUMN-REVERSE,
    }

    resource element{
        constructor(element-type:element-type,content:option<string>);
        content:func(content:option<string>) -> element;

        flex:func() -> element;
        flex-direction:func(direction:flex-direction) -> element;

        margin:func(margin:u32) -> element;
        margin-top:func(margin:u32) -> element;
        margin-bottom:func(margin:u32) -> element;
        margin-left:func(margin:u32) -> element;
        margin-right:func(margin:u32) -> element;

        padding:func(padding:u32) -> element;
        padding-top:func(padding:u32) -> element;
        padding-bottom:func(padding:u32) -> element;
        padding-left:func(padding:u32) -> element;
        padding-right:func(padding:u32) -> element;

        align-center:func() -> element;
        align-end:func() -> element;
        align-start:func() -> element;

        justify-center:func() -> element;
        justify-start:func() -> element;
        justify-end:func() -> element;

        bg:func(color:string) -> element;
        text-color:func(color:string) -> element;

        size:func(size:u32) -> element;
        width:func(width:u32) -> element;
        width-full:func() -> element;
        width-half:func() -> element;
        height:func(height:u32) -> element;
        height-full:func() -> element;
        height-half:func() -> element;
        radius:func(radius:u32) -> element;
        border:func(width:u32,color:string) -> element;

        relative:func() -> element;
        absolute:func() -> element;

        top:func(position:u32) -> element;
        bottom:func(position:u32) -> element;
        left:func(position:u32) -> element;
        right:func(position:u32) -> element;

        opacity:func(opacity:f32) -> element;
        transition:func(transition:string) -> element;
        without-default-styles:func() -> element;

        z-index:func(z:s32) -> element;
        disabled:func() -> element;

        child:func(child:element) -> element;

        on:func(event:event,id:string) -> element;
    }

    enum event {
        CLICK,
        HOVER,
        CHANGE,
        INPUT,
        FOCUS,
        BLUR,
        MOUSE-ENTER,
        MOUSE-LEAVE,
        POINTER-DOWN,
        POINTER-UP,
        POINTER-MOVE,
    }

    render:func(id:string,el:element);
    render-to-text-card:func(id:string,text:string);
}
```

## 核心说明

- `render(id, el)` 用于渲染插件页面主体，或者渲染一个已经通过 `register::register-card(CardType::ELEMENT, ...)` 注册的元素卡片。
- `render-to-text-card(id, text)` 用于渲染已经注册的 `TEXT` 卡片。
- 旧版 `ui` 只有基础元素和样式能力，没有 `gap`、`grid`、`scroll-area`、`tabs`、`dialog`、动画扩展等能力。

## 注意事项

暂无

## 元素类型

- `BUTTON`、`INPUT`、`SELECT`、`OPTION`
- `IMAGE`、`VIDEO`、`AUDIO`、`SVG`
- `DIV`、`SPAN`、`P`

## 事件类型

- `CLICK`
- `HOVER`
- `CHANGE`
- `INPUT`
- `FOCUS`
- `BLUR`
- `MOUSE-ENTER`
- `MOUSE-LEAVE`
- `POINTER-DOWN`
- `POINTER-UP`
- `POINTER-MOVE`

## 链式方法

### 创建与布局

- `element::new(element-type, content)`：创建元素。
- `content(content)`：更新内容。
- `flex()` / `flex-direction(direction)`：开启 Flex 布局并设置方向。
- `child(child)`：追加子元素。

### 间距与对齐

- `margin` / `margin-top` / `margin-bottom` / `margin-left` / `margin-right`
- `padding` / `padding-top` / `padding-bottom` / `padding-left` / `padding-right`
- `align-start` / `align-center` / `align-end`
- `justify-start` / `justify-center` / `justify-end`

### 尺寸与视觉

- `size` / `width` / `width-full` / `width-half`
- `height` / `height-full` / `height-half`
- `bg` / `text-color`
- `radius` / `border`
- `opacity` / `transition`
- `without-default-styles`

### 定位与状态

- `relative` / `absolute`
- `top` / `bottom` / `left` / `right`
- `z-index`
- `disabled`
- `on(event, id)`

## Rust 示例

```rust tab="Rust"
use crate::astrobox::psys_host::ui::{self, ElementType, Event, FlexDirection};

pub fn render_legacy_card(element_id: &str) {
    let button = ui::element::new(ElementType::Button, Some("开始同步".into()))
        .padding(10)
        .radius(8)
        .bg("#2B5BE8")
        .text_color("#FFFFFF")
        .on(Event::Click, "start-sync");

    let root = ui::element::new(ElementType::Div, None)
        .flex()
        .flex_direction(FlexDirection::Column)
        .padding(16)
        .child(button);

    ui::render(element_id, root);
}
```

```go tab="Go"
package plugin

import (
	ui "astroboxplugin/bindings/astrobox_psys_host_ui"

	"github.com/bytecodealliance/wit-bindgen/wit_types"
)

func renderLegacyCard(elementID string) {
	button := ui.MakeElement(ui.ElementTypeButton, wit_types.Some("开始同步")).
		Padding(10).
		Radius(8).
		Bg("#2B5BE8").
		TextColor("#FFFFFF").
		On(ui.EventClick, "start-sync")

	root := ui.MakeElement(ui.ElementTypeDiv, wit_types.None[string]()).
		Flex().
		FlexDirection(ui.FlexDirectionColumn).
		Padding(16).
		Child(button)

	ui.Render(elementID, root)
}
```
