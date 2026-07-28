---
title: Manifest 文件
---

## 什么是 Manifest 文件

**Manifest 文件**是 AstroBox 插件的**核心描述文件**，用于声明插件的基本信息、加载方式、运行环境要求以及所需权限。

AstroBox 在加载插件时，会首先解析 Manifest 文件，并根据其中的配置决定：

- 插件如何被识别与展示  
- 插件的入口文件  
- 插件可使用的 WASI / AstroBox API 能力  

如果 Manifest 文件缺失或格式不正确，插件将无法被加载。

---

## Manifest 结构定义（Rust）

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PluginManifest {
    pub name: String,             // 插件名称
    pub icon: String,             // 插件图标（路径）
    pub version: String,          // 插件版本
    pub description: String,      // 插件简介
    pub author: String,           // 插件作者
    pub website: String,          // 插件网站（例如 GitHub 仓库地址）
    pub entry: String,            // 插件入口 wasm 文件
    pub wasi_version: u32,        // WASI 接口版本
    pub api_level: u32,           // 插件 API 等级
    pub permissions: Vec<String>, // 插件权限列表
    #[serde(default)]
    pub additional_files: Vec<String>, // 插件附加文件列表
}
```

---

## 字段说明

| 字段名 | 类型 | 是否必填 | 说明 |
|------|----|--------|----|
| name | String | 是 | 插件名称 |
| icon | String | 是 | 插件图标路径（相对插件根目录） |
| version | String | 是 | 插件版本号 |
| description | String | 是 | 插件的简要说明 |
| author | String | 是 | 插件作者 |
| website | String | 是 | 插件主页或源码仓库地址 |
| entry | String | 是 | 插件入口 WASM 文件 |
| wasi_version | u32 | 是 | WASI 接口版本 |
| api_level | u32 | 是 | AstroBox API 等级 |
| permissions | Vec<String> | 是 | 插件所需权限列表 |
| additional_files | Vec<String> | 否 | 插件运行所需的额外文件（填随插件包体上传的即可） |

---

## 字段详细说明

### name
插件的显示名称，建议简短清晰

### icon
插件图标路径，相对插件根目录，推荐使用 png，不建议使用 svg

### version
插件版本号，推荐使用语义化版本（如 1.0.0）

### description
插件的功能简介，用于帮助用户快速理解插件用途

### author
插件作者名称

### website
插件主页地址，例如 GitHub 仓库或项目官网

### entry
插件入口 WASM 文件路径，AstroBox 从此文件开始加载插件

### wasi_version
插件所依赖的 WASI 接口版本，用于运行时兼容判断，例如 2 对应 wasi-p2

### api_level
插件所使用的 AstroBox API 等级，向下兼容，在 [此处](./apilevel#三api-level-与-astrobox-版本对应关系) 查看每个 API Level 所对应的 AstroBox 版本

### permissions
插件运行所需权限列表，未声明的权限将被拒绝访问

示例：

```json
"permissions": [
    "device",
    "interconnect"
]
```

### additional_files（可选）
插件运行所需的附加文件列表，只需要填写随插件包体上传的文件，以便在下载插件时一起下载并进行大小计算。如果你只通过abp分发插件而不上架 AstroBox 官方插件源，则无需这么做

---

## 完整示例

```json
{
  "name": "Hello AstroBox",
  "icon": "icon.png",
  "version": "1.0.0",
  "description": "一个示例 AstroBox 插件",
  "author": "AstroBox Team",
  "website": "https://github.com/example/astrobox-plugin",
  "entry": "plugin.wasm",
  "wasi_version": 1,
  "api_level": 1,
  "permissions": [
    "network"
  ],
  "additional_files": [
    "extra_tools.rpk"
  ]
}
```

---

## 注意事项

- Manifest 文件必须是合法 json
- 缺失必填字段将导致插件加载失败
- 权限声明应遵循最小权限原则
