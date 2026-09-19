---
title: Python 上手
---

## 准备

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install componentize-py==0.25.1
```

从模板开始：

```bash
git clone --recurse-submodules https://github.com/AstralSightStudios/AstroBox-NG-Plugin-Template-Python
cd AstroBox-NG-Plugin-Template-Python
```

## 写插件

componentize-py 按 WIT 里的**导出接口名**在入口模块里找实现类，所以类名必须是
`Lifecycle` 和 `Event`：

```python
import wit_world.exports as exports
from wit_world.imports import os as host_os, ui


class Lifecycle(exports.Lifecycle):
    async def on_load(self) -> None:
        # on-load 是 async func，直接 await 宿主接口
        arch = await host_os.arch()
        print(f"hello from {arch}")


class Event(exports.Event):
    async def on_event(self, event_type, event_payload: str) -> str:
        return ""

    async def on_ui_event(self, event_id: str, event, event_payload: str) -> str:
        return ""

    async def on_ui_render(self, element_id: str) -> None:
        size = await ui.get_render_size()
        root = ui.Element(ui.ElementType.DIV, None).flex()
        ui.render(element_id, root.child(ui.Element(ui.ElementType.P, f"{size.width}px")))

    async def on_card_render(self, card_id: str) -> None:
        ui.render_to_text_card(card_id, "hello")
```

类名写错会报：

```
TypeError: Can't instantiate abstract class Lifecycle without an implementation for abstract method 'on_load'
```

## 构建

```bash
python scripts/build_dist.py --package
```

也可以直接调 componentize-py：

```bash
componentize-py -d wit -w psys-world-v4 componentize app -p src -o plugin.wasm
```

## 需要知道的

- **产物约 20 MB**（内嵌 CPython）。宿主首次加载会预编译并缓存，之后启动很快，
  但插件包体积确实比 Rust 大一个量级。
- **运行时导入必须在模块顶层解析完**。componentize-py 只在构建期解析依赖，
  `import x` 之后再用 `x.y.foo()` 可能找不到 `y`，要写成 `from x import y`。
- 生成绑定（给 IDE 用）：`componentize-py -d wit -w psys-world-v4 bindings gen`
