# EeveeSpotify 分析报告

## 1. 项目核心作用
`EeveeSpotify` 是一个 iOS 越狱插件，旨在为非订阅用户解锁 Spotify Premium 体验。其核心功能包括：
- **付费功能解锁**：去除广告、启用无限切歌、开启“随点随播”（On-Demand）。
- **增强歌词**：通过集成第三方歌词源（Genius, Musixmatch, LRCLIB, PetitLyrics）替换原有的歌词限制。
- **自定义配置**：提供丰富的设置选项，允许用户调整 UI 和歌词显示效果。

## 2. 实现原理深度剖析

项目采用 **动态请求拦截与数据篡改 (Dynamic Request Interception & Data Modification)** 的技术路径。

### A. 网络层 Hook
- **框架**：利用 **Orion** 框架挂钩 Spotify 内部的 `SPTDataLoaderService` 类。
- **拦截点**：在 `DataLoaderServiceHooks.x.swift` 中，通过 Hook `URLSession` 的数据接收回调，实时捕获 Spotify 与服务器间的通信。

### B. Protobuf 数据修改
Spotify 使用 **Protobuf (Protocol Buffers)** 传输用户信息和配置。
- **反序列化**：当捕获到特定的 URL（如 `/user-customization-service/`）时，插件将二进制数据解析为对应的 Swift 模型。
- **静态注入**：在 `DynamicPremium+ModifyingFunctions.swift` 中，强制修改以下关键属性：
    - 将 `ads` (广告) 设为 `false`。
    - 将 `player-license` (播放许可) 设为 `premium`。
    - 将 `catalogue` (曲库状态) 设为 `premium`。
    - 将 `on-demand` (随点随播) 设为 `true`。
- **重写响应**：修改后的模型被重新序列化为 Protobuf 二进制格式，伪造成原始服务器响应返回给客户端。

### C. 歌词注入逻辑
- **拦截原始请求**：拦截 Spotify 的官方歌词 API 请求。
- **第三方调用**：根据当前播放的歌曲元数据（歌手、歌名），异步调用第三方歌词 API（如 Genius）。
- **数据封装**：将第三方返回的文本封装进 Spotify 预期的 `Lyrics` 数据结构中，模拟官方歌词的显示效果，从而突破次数限制。

## 3. 关键组件与目录说明
- `Sources/EeveeSpotify/Premium/`: 核心 Premium 补丁代码，负责修改各类功能开关。
- `Sources/EeveeSpotify/Lyrics/`: 歌词模块，包含各大歌词源的爬取和解析逻辑。
- `Sources/EeveeSpotify/Tweak.x.swift`: 插件的初始化逻辑，根据 Spotify 版本加载不同的 Hook 策略。
- `layout/`: 包含插件的国际化资源和偏好设置图标。

---
*注：该分析基于项目源代码及 README 描述。*
