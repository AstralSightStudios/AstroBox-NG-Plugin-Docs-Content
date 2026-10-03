---
title: JavaScript 上手
---

## 准备

JS 插件需要**我们打过补丁的 ComponentizeJS**（原因见[语言选择](./language)）。
模板里已经配好，只需要本机有 Node 与 Rust：

```bash
rustup target add wasm32-wasip1   # npm 安装 git 依赖时要现场构建 splicer
```

```bash
git clone --recurse-submodules https://github.com/AstralSightStudios/AstroBox-NG-Plugin-Template-JS
cd AstroBox-NG-Plugin-Template-JS
npm install
```

## 写插件

导出对象名对应 WIT 里的接口名（`lifecycle` / `event`）：

```js
import { arch, platform } from 'astrobox:psys-host-v4/os';
import { render, getRenderSize, Element } from 'astrobox:psys-host-v4/ui';

export const lifecycle = {
  async onLoad() {
    console.log(`hello from ${platform()}/${arch()}`);
  },
};

export const event = {
  async onEvent(eventType, payload) { return ''; },
  async onUiEvent(eventId, uiEvent, payload) { return ''; },
  async onUiRender(elementId) {
    const size = getRenderSize();
    const root = new Element('div', null).flex();
    render(elementId, root.child(new Element('p', `${size.width}px`)));
  },
  async onCardRender(cardId) {},
};
```

## 两个容易踩的点

**WIT 的 enum 在 JS 里是字符串。** 没有 `ElementType.DIV` 这种东西：

```js
new Element('div', 'text')        // ✅
new Element(ElementType.DIV, ...)  // ❌ 会报 doesn't provide an export named 'ElementType'
```

**宿主接口是同步调用的。** 虽然 WIT 里写着 `async func`，但被同步 lower 了，
JS 侧直接当普通函数用，调用期间线程阻塞到宿主返回：

```js
const size = getRenderSize();   // 不需要 await
```

导出函数可以是 `async`，引擎会在返回前把 Promise 结算掉。

## 构建

```bash
npm run build      # -> dist/plugin.wasm
npm run package    # 顺便打 .abp
```

产物约 14 MB（内嵌 SpiderMonkey）。宿主首次加载会预编译并缓存。
