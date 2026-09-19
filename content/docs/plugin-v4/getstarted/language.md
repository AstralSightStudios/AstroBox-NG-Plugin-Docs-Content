---
title: 语言选择
---

Level 4 的接口是 `async func`，因此**能不能用某门语言，取决于它的工具链是否实现了
组件模型的 async**。下面是实测结论，不是推测。

| 语言 | 状态 | 说明 |
|----|----|----|
| **Rust** | ✅ 官方支持 | `wit-bindgen` 对 async 支持最完整，性能与体积最好 |
| **Python** | ✅ 官方支持 | `componentize-py` 0.25+ 原生支持 async |
| **JavaScript** | ✅ 官方支持 | 需用我们打过补丁的 ComponentizeJS，见下 |
| 其他 | ⚠️ 自行验证 | 取决于该语言绑定生成器对 async 的支持程度 |

## JavaScript 的特殊情况

上游 ComponentizeJS（jco 用的 SpiderMonkey 嵌入方案）到 0.22 为止**没有实现
`async func`**：WIT 里只要出现一个，splicer 就会直接 panic：

```
thread '<unnamed>' panicked at crates/spidermonkey-embedding-splicer/src/bindgen.rs:719:
not yet implemented
```

我们维护了一个分支把这块补上：
[`AstralSightStudios/ComponentizeJS@astrobox/async-func-support`](https://github.com/AstralSightStudios/ComponentizeJS/tree/astrobox/async-func-support)

做法是把 async 函数按**同步 lower / 同步 lift** 处理。这在组件模型里是合法的——
`async` 描述的是「被调方可能阻塞」这一**类型属性**，并不要求调用方也用 async ABI；
而运行时判断能否阻塞看的是类型而非 lift 方式（wasmtime 的规则是
`may_block = 类型是 async || 已返回`），所以同步 lift 的导出照样可以阻塞等待宿主。

JS 插件模板已经指向该分支，正常 `npm install` 即可。上游合入后会切回官方包。

### 对写代码的影响

宿主接口虽然在 WIT 里是 `async func`，但在 JS 侧是**同步调用**：

```js
import { arch } from 'astrobox:psys-host-v4/os';

// 不需要 await，调用期间 JS 线程阻塞到宿主返回
const a = arch();
```

导出函数则可以照常写成 `async`，引擎会在返回前结算 Promise。
