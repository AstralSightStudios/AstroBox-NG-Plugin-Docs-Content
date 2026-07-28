---
title: UI 接口 (API Level 3+)
---

`ui-v3` 是当前推荐使用的 UI Builder 接口。它提供了比旧版 `ui` 更完整的元素类型、布局能力和动画能力。

## 接口定义

```wit
interface ui-v3 {
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
        TEXTAREA,
        SWITCH,
        SLIDER,
        PROGRESS,
        GRID,
        SCROLL-AREA,
        LIST,
        LIST-ITEM,
        CODE,
        CARD,
        TABS-ROOT,
        TABS-LIST,
        TABS-TRIGGER,
        TABS-CONTENT,
        ICON,
        DIVIDER,
        CONTEXT-MENU-ROOT,
        CONTEXT-MENU-TRIGGER,
        CONTEXT-MENU-CONTENT,
        CONTEXT-MENU-ITEM,
        CONTEXT-MENU-SEPARATOR,
        DIALOG-ROOT,
        DIALOG-TRIGGER,
        DIALOG-CONTENT,
        DIALOG-TITLE,
        DIALOG-DESCRIPTION,
        DIALOG-CLOSE,
        DROPDOWN-MENU-ROOT,
        DROPDOWN-MENU-TRIGGER,
        DROPDOWN-MENU-CONTENT,
        DROPDOWN-MENU-ITEM,
        DROPDOWN-MENU-SEPARATOR,
        TOOLTIP,
        CHECKBOX,
        SEPARATOR,
        BADGE,
        ALERT-DIALOG-ROOT,
        ALERT-DIALOG-TRIGGER,
        ALERT-DIALOG-CONTENT,
        ALERT-DIALOG-TITLE,
        ALERT-DIALOG-DESCRIPTION,
        ALERT-DIALOG-ACTION,
        ALERT-DIALOG-CANCEL,
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
        transform:func(value:string) -> element;
        transform-origin:func(value:string) -> element;
        animation:func(value:string) -> element;
        animation-name:func(name:string) -> element;
        animation-duration-ms:func(ms:u32) -> element;
        animation-delay-ms:func(ms:u32) -> element;
        animation-easing:func(easing:string) -> element;
        animation-iteration-count:func(count:string) -> element;
        animation-direction:func(direction:string) -> element;
        animation-fill-mode:func(fill-mode:string) -> element;
        animation-play-state:func(play-state:string) -> element;
        animation-preset:func(name:string) -> element;
        will-change:func(value:string) -> element;
        filter:func(value:string) -> element;
        backdrop-filter:func(value:string) -> element;
        perspective:func(value:string) -> element;
        backface-visibility:func(value:string) -> element;
        without-default-styles:func() -> element;
        autofocus:func() -> element;
        tab-index:func(index:s32) -> element;

        z-index:func(z:s32) -> element;
        disabled:func() -> element;

        child:func(child:element) -> element;

        on:func(event:event,id:string) -> element;

        grid-template-columns:func(columns:string) -> element;
        gap:func(spacing:u32) -> element;

        max-width:func(width:u32) -> element;
        max-height:func(height:u32) -> element;
        min-width:func(width:u32) -> element;
        min-height:func(height:u32) -> element;
        scroll-top:func(position:u32) -> element;
        scroll-left:func(position:u32) -> element;
        scroll-to:func(top:u32, left:u32) -> element;
        scroll-behavior:func(behavior:string) -> element;

        flex-grow:func(value:f32) -> element;
        flex-shrink:func(value:f32) -> element;
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
        KEY-DOWN,
        KEY-UP,
        LONG-PRESS,
    }

    render:func(id:string,el:element);
    render-to-text-card:func(id:string,text:string);
}
```

## 元素类型

- 基础输入与展示：`BUTTON`、`INPUT`、`TEXTAREA`、`SELECT`、`OPTION`、`SWITCH`、`SLIDER`、`PROGRESS`
- 媒体与文本：`IMAGE`、`VIDEO`、`AUDIO`、`SVG`、`DIV`、`SPAN`、`P`、`CODE`
- 布局：`GRID`、`SCROLL-AREA`、`LIST`、`LIST-ITEM`、`CARD`
- 选项卡：`TABS-ROOT`、`TABS-LIST`、`TABS-TRIGGER`、`TABS-CONTENT`
- 菜单与提示：`ICON`、`DIVIDER`、`TOOLTIP`、`CHECKBOX`、`SEPARATOR`、`BADGE`
- Context Menu：`CONTEXT-MENU-ROOT`、`CONTEXT-MENU-TRIGGER`、`CONTEXT-MENU-CONTENT`、`CONTEXT-MENU-ITEM`、`CONTEXT-MENU-SEPARATOR`
- Dialog：`DIALOG-ROOT`、`DIALOG-TRIGGER`、`DIALOG-CONTENT`、`DIALOG-TITLE`、`DIALOG-DESCRIPTION`、`DIALOG-CLOSE`
- Dropdown Menu：`DROPDOWN-MENU-ROOT`、`DROPDOWN-MENU-TRIGGER`、`DROPDOWN-MENU-CONTENT`、`DROPDOWN-MENU-ITEM`、`DROPDOWN-MENU-SEPARATOR`
- Alert Dialog：`ALERT-DIALOG-ROOT`、`ALERT-DIALOG-TRIGGER`、`ALERT-DIALOG-CONTENT`、`ALERT-DIALOG-TITLE`、`ALERT-DIALOG-DESCRIPTION`、`ALERT-DIALOG-ACTION`、`ALERT-DIALOG-CANCEL`

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
- `KEY-DOWN`
- `KEY-UP`
- `LONG-PRESS`

## 链式方法

### 创建与结构

- `element::new(element-type, content)`
- `content(content)`
- `child(child)`
- `on(event, id)`

### Flex / Grid / Scroll

- `flex()` / `flex-direction(direction)`
- `grid-template-columns(columns)`
- `gap(spacing)`
- `scroll-top(position)` / `scroll-left(position)` / `scroll-to(top, left)` / `scroll-behavior(behavior)`
- `flex-grow(value)` / `flex-shrink(value)`

### 间距与对齐

- `margin` / `margin-top` / `margin-bottom` / `margin-left` / `margin-right`
- `padding` / `padding-top` / `padding-bottom` / `padding-left` / `padding-right`
- `align-start` / `align-center` / `align-end`
- `justify-start` / `justify-center` / `justify-end`

### 尺寸

- `size`
- `width` / `width-full` / `width-half`
- `height` / `height-full` / `height-half`
- `max-width` / `max-height`
- `min-width` / `min-height`

### 视觉与动画

- `bg` / `text-color`
- `radius` / `border`
- `opacity` / `transition`
- `transform` / `transform-origin`
- `animation` / `animation-name`
- `animation-duration-ms` / `animation-delay-ms`
- `animation-easing` / `animation-iteration-count`
- `animation-direction` / `animation-fill-mode` / `animation-play-state`
- `animation-preset`
- `will-change`
- `filter` / `backdrop-filter`
- `perspective` / `backface-visibility`

### 定位与交互

- `relative` / `absolute`
- `top` / `bottom` / `left` / `right`
- `z-index`
- `without-default-styles`
- `autofocus`
- `tab-index`
- `disabled`

## 渲染说明

- `render(id, el)`：渲染到插件页面主体，或者渲染到一个已注册的 `ELEMENT` 卡片。
- `render-to-text-card(id, text)`：渲染到已注册的 `TEXT` 卡片。
- 如果插件实现了 `on-ui-render(element-id)` / `on-card-render(card-id)`，宿主会把需要渲染的目标 ID 传给你，你再用这个 ID 调用 `render` 或 `render-to-text-card`。

## 注意事项

- 不可与旧版ui混用

## 示例

```rust tab="Rust"
use crate::astrobox::psys_host::ui_v3::{self, ElementType, Event, FlexDirection};

pub fn render_dashboard(element_id: &str) {
    let status = ui_v3::element::new(ElementType::Badge, Some("已连接".into()))
        .padding(6)
        .radius(999)
        .bg("#163B2C")
        .text_color("#87E9C6");

    let refresh = ui_v3::element::new(ElementType::Button, Some("刷新设备".into()))
        .padding(10)
        .radius(10)
        .bg("#11182C")
        .text_color("#FFFFFF")
        .on(Event::Click, "device-refresh");

    let root = ui_v3::element::new(ElementType::Div, None)
        .flex()
        .flex_direction(FlexDirection::Column)
        .gap(12)
        .padding(12)
        .child(status)
        .child(refresh);

    ui_v3::render(element_id, root);
}
```

```go tab="Go"
package plugin

import (
	ui "astroboxplugin/bindings/astrobox_psys_host_ui_v3"

	"github.com/bytecodealliance/wit-bindgen/wit_types"
)

func renderDashboard(elementID string) {
	status := ui.MakeElement(ui.ElementTypeBadge, wit_types.Some("已连接")).
		Padding(6).
		Radius(999).
		Bg("#163B2C").
		TextColor("#87E9C6")

	refresh := ui.MakeElement(ui.ElementTypeButton, wit_types.Some("刷新设备")).
		Padding(10).
		Radius(10).
		Bg("#11182C").
		TextColor("#FFFFFF").
		On(ui.EventClick, "device-refresh")

	root := ui.MakeElement(ui.ElementTypeDiv, wit_types.None[string]()).
		Flex().
		FlexDirection(ui.FlexDirectionColumn).
		Gap(12).
		Padding(12).
		Child(status).
		Child(refresh)

	ui.Render(elementID, root)
}
```
