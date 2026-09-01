# PRODUCT.md

> Version: 0.1.0  
> Status: Draft  
> Product: Fluvra  
> Last updated: 2026-09-01

---

# 1. Product Overview

Fluvra 是一个面向 Windows 和 macOS 的现代媒体下载应用。

它的目标不是成为 yt-dlp 的图形界面，也不是向用户暴露大量命令行参数，而是把链接解析、媒体选择、下载管理、字幕、音轨、后处理、历史记录和站点兼容性封装成一个简单、可靠的桌面产品。

用户应该能够完成：

> 发现一个视频 → 将链接交给 Fluvra → 得到自己想要的文件。

底层具体使用什么解析器、下载器、媒体处理工具，不应成为普通用户需要理解的内容。

Fluvra 可以由 yt-dlp、FFmpeg、FFprobe、JavaScript Runtime 等成熟开源组件驱动，但这些组件属于实现细节，而不是产品本身。

---

# 2. Product Positioning

## 2.1 Product Definition

Fluvra 是：

> **A modern media downloader for Windows and macOS.**

核心价值：

- 简单
- 可靠
- 快速
- 可恢复
- 可解释
- 跨平台
- 长期可维护

---

## 2.2 What Fluvra Is Not

Fluvra 不是：

- yt-dlp 参数生成器
- FFmpeg GUI
- 通用视频剪辑软件
- 专业视频转码工作站
- 在线视频网站
- 内容搜索平台
- DRM 破解工具
- 视频内容聚合平台
- 云端视频下载服务

Fluvra 的核心始终是：

> **把用户有权访问的网络媒体可靠地保存为本地文件。**

---

# 3. Product Vision

Fluvra 的长期目标是达到甚至超过 Downie 一类成熟商业下载器的日常体验，同时提供：

- Windows + macOS 跨平台支持
- 更强的下载历史与本地媒体库
- 更透明的下载任务状态
- 更完整的 FFmpeg 后处理能力
- 独立于应用版本的下载核心更新
- 更好的错误解释与恢复机制
- 可扩展的网站 Integration 系统
- 更开放的 GitHub 社区生态

最终，用户无需知道一个网站背后使用的是：

- yt-dlp
- Direct Media
- HLS
- DASH
- 自定义 Integration
- Smart Capture

用户只需要知道：

> Fluvra 可以把这个媒体正确下载下来。

---

# 4. Product Principles

Fluvra 的所有产品与工程决策都应遵守以下原则。

## 4.1 Hide Implementation Complexity

用户不需要理解：

- yt-dlp
- FFmpeg
- codec
- container
- format ID
- DASH
- HLS
- cookies.txt
- command-line arguments

普通用户应通过自然的产品概念完成任务，例如：

- 最佳质量
- 高兼容性
- 仅音频
- 下载字幕
- 下载到指定文件夹

高级技术选项只能存在于高级设置中。

---

## 4.2 Correct Defaults First

默认行为应该适合绝大多数用户。

用户粘贴链接后，不应该必须完成多个配置步骤才能开始下载。

例如：

默认：

- 自动选择最佳合理质量
- 自动合并视频和音频
- 自动选择适合的容器
- 自动生成安全文件名
- 自动处理必要的媒体后处理

只有用户主动要求时，才需要进一步配置。

---

## 4.3 Reliable Tasks

下载任务必须被视为长期存在的任务对象，而不是一个临时命令行进程。

每个任务应该：

- 有明确状态
- 可以追踪进度
- 可以取消
- 可以失败
- 可以重试
- 可以恢复
- 可以查看错误原因
- 可以找到最终文件

应用异常退出后，不应无条件丢失所有任务信息。

---

## 4.4 Explain Failures

Fluvra 不应该直接把底层工具错误原样交给普通用户。

例如不应该只显示：

`ERROR: Sign in to confirm you're not a bot`

而应该解释：

> 此媒体需要登录后才能访问。

并提供下一步：

> 使用浏览器登录

底层日志仍然可以保留给高级用户和问题诊断。

---

## 4.5 Core Independent From App Lifecycle

视频网站会持续变化。

因此：

- App
- Download Core
- FFmpeg
- JavaScript Runtime
- Integrations

必须允许独立更新。

一个网站发生变化，不应该强制 Fluvra 发布完整的新桌面应用版本才能恢复下载。

---

## 4.6 Generic First, Site-Specific Second

优先使用通用解析能力。

只有通用能力无法解决时，才增加：

- Site Integration
- Special Resolver
- Smart Capture

避免把大量网站特殊逻辑写死进主应用。

---

## 4.7 DRM Is Out of Scope

Fluvra 不以绕过 DRM 为目标。

明确不将以下能力作为产品功能：

- Widevine 破解
- FairPlay 破解
- DRM key 提取
- 付费内容保护绕过

Fluvra 只处理用户能够正常访问并且不存在受保护 DRM 限制的媒体内容。

---

# 5. Target Platforms

首要支持平台：

## Windows

Initial target:

- Windows 11
- x86-64

未来可考虑：

- Windows ARM64

---

## macOS

Initial target:

- Apple Silicon
- macOS ARM64

未来根据需要考虑：

- Intel macOS
- Universal Binary

---

# 6. Target Users

## 6.1 Primary User

希望保存在线视频，但不想学习命令行工具的普通用户。

他们可能：

- 知道视频链接
- 知道自己想要 1080p / 4K
- 知道自己想保存 MP4 或音频
- 不知道 yt-dlp 是什么
- 不知道 format ID 是什么
- 不关心视频音频为什么需要合并

---

## 6.2 Power User

了解媒体格式或者 yt-dlp，希望获得更强控制能力的高级用户。

可能需要：

- 指定容器
- 指定 codec
- 自定义字幕
- 自定义文件名
- Proxy
- Browser Cookies
- 自定义 yt-dlp 参数
- 自定义后处理
- 查看完整日志

高级能力不能破坏普通模式的简洁性。

---

# 7. Primary User Scenarios

## 7.1 Paste and Download

用户复制一个视频地址。

打开 Fluvra。

按：

- `Ctrl + V`
- `Command + V`

Fluvra：

1. 自动读取 URL
2. 开始解析
3. 显示视频信息
4. 自动选择合理配置
5. 用户点击 Download
6. 加入下载队列

---

## 7.2 Drag and Drop

用户把：

- URL
- 文本
- 浏览器链接

拖入 Fluvra。

应用自动识别并解析。

---

## 7.3 Download Best Quality

用户只想：

> 下载最好的版本。

不需要理解：

- VP9
- AV1
- H.264
- Opus
- AAC
- MKV
- MP4

Fluvra 自动选择。

---

## 7.4 Download Compatible Video

用户希望视频在绝大多数设备上可以直接播放。

选择：

> High Compatibility

Fluvra 自动选择或者转换成合理的：

- MP4
- H.264
- AAC

具体实现由 Media Engine 决定。

---

## 7.5 Audio Only

用户希望只保存音频。

用户选择：

> Audio

然后可以选择：

- Best Audio
- M4A
- MP3
- FLAC

---

## 7.6 Download Playlist

当 URL 包含 Playlist 时：

Fluvra 应识别 Playlist，并允许：

- 下载全部
- 选择部分
- 仅下载当前视频

---

## 7.7 Download Subtitles

如果存在字幕：

Fluvra 应显示：

- 字幕语言
- 自动字幕与人工字幕的差异
- 是否下载
- 是否嵌入文件

---

## 7.8 Previously Downloaded Content

如果用户再次添加已经下载过的媒体：

Fluvra 应提醒：

> This media was downloaded before.

并提供：

- Show File
- Open Source
- Download Again
- Cancel

---

## 7.9 Failed Download

下载失败时：

Fluvra 应：

1. 分类错误
2. 显示用户可理解的原因
3. 提供合理恢复操作

例如：

> Login required

操作：

> Sign In

或者：

> Download Core may be outdated

操作：

> Update Download Core

---

# 8. Main Navigation

Fluvra 默认只提供四个一级页面：

- Home
- Downloads
- Library
- Settings

避免按照技术能力拆成：

- Video
- Audio
- Converter
- Playlist
- Cookies
- FFmpeg
- yt-dlp
- Advanced
- Logs

技术复杂度不应该成为信息架构。

---

# 9. Home

Home 是媒体进入应用的主要入口。

核心区域：

> Drop or paste a media link

支持：

- Paste URL
- Drag URL
- Recent clipboard URL
- Browser Extension entry

解析完成后显示：

- Thumbnail
- Title
- Creator
- Duration
- Source website
- Quality
- Preset
- Audio
- Subtitle
- Output location

主要操作：

> Download

次要操作：

> Add to Queue

---

# 10. Downloads

Downloads 是当前任务管理中心。

任务状态至少包括：

- Resolving
- Pending
- Waiting
- Downloading
- Processing
- Completed
- Failed
- Cancelled

用户看到的信息包括：

- Thumbnail
- Title
- Progress
- Download Speed
- Downloaded Size
- Estimated Remaining Time
- Current Stage

例如：

> Downloading Video

> Downloading Audio

> Merging

> Embedding Subtitles

> Processing Metadata

操作：

- Pause（后续视底层能力）
- Cancel
- Retry
- Show File
- Open Folder
- Remove

---

# 11. Library

Library 不只是简单 History。

它是用户下载内容的本地数据库。

至少记录：

- Original URL
- Canonical URL
- Website
- Media ID
- Title
- Creator
- Thumbnail
- Duration
- Download Time
- Output Path
- File Size
- Resolution
- Container
- Video Codec
- Audio Codec
- Subtitle Languages
- Preset

Library 应支持：

- Search
- Website filter
- Date filter
- Creator filter
- Download Again
- Show File
- Open Source
- Remove History Entry

后续可增加：

- Playlist grouping
- Tags
- Favorites
- Smart Collections

---

# 12. Presets

Fluvra 不应要求普通用户配置底层格式规则。

提供以下默认 Preset：

## Best Quality

目标：

> 获取最高合理质量。

自动选择：

- 最佳视频
- 最佳音频
- 合理容器

---

## High Compatibility

目标：

> 在大多数设备和播放器中直接播放。

优先：

- MP4
- H.264
- AAC

必要时允许 FFmpeg 转换。

---

## Audio

目标：

> 只保留音频。

默认：

- Best Audio

可以进一步选择：

- M4A
- MP3
- FLAC

---

## Archive

目标：

> 尽可能完整保存媒体信息。

包括：

- Highest Quality
- Audio
- Subtitles
- Metadata
- Thumbnail
- Chapters

---

## Custom

高级用户模式。

允许进一步控制：

- Resolution
- Codec
- Container
- Audio
- Subtitle
- Metadata
- Post-processing

---

# 13. Download Queue

Fluvra 必须提供真正的下载队列。

队列需要支持：

- 多任务
- 并发数量限制
- FIFO
- Retry
- Cancel
- Reordering（Phase 2）
- Priority（Phase 2）

应用退出和再次启动时，应尽可能保留未完成任务状态。

---

# 14. Media Processing

下载后的媒体处理属于 Fluvra 的核心能力。

用户不应该需要另一个转换软件完成常见操作。

需要支持：

- Merge video + audio
- Remux
- Audio extraction
- Subtitle embedding
- Thumbnail embedding
- Metadata embedding
- Chapters
- Format conversion

高级视频编辑不属于 Fluvra 的核心范围。

例如不优先提供：

- Timeline editing
- Filters
- Color grading
- Complex encoding profiles

---

# 15. Error Handling

所有底层错误都应该被转换为产品层错误。

初期错误类别包括：

- NETWORK_ERROR
- TIMEOUT
- AUTH_REQUIRED
- LOGIN_EXPIRED
- UNSUPPORTED_URL
- MEDIA_UNAVAILABLE
- GEO_RESTRICTED
- RATE_LIMITED
- CORE_OUTDATED
- DISK_FULL
- FILE_PERMISSION
- POSTPROCESS_FAILED
- DEPENDENCY_MISSING
- CANCELLED
- UNKNOWN_ERROR

每种错误应定义：

- 用户可读标题
- 简单解释
- 建议操作
- 技术详情

---

# 16. Diagnostics

Fluvra 应内置 Diagnostics 页面。

至少检查：

- Application Version
- Download Core Version
- FFmpeg Version
- FFprobe Version
- JavaScript Runtime Version
- Integration Registry Version
- Download Directory
- Write Permission
- Free Disk Space

用户可以：

> Run Diagnostics

高级模式可以：

> Export Diagnostic Report

用于 GitHub Issues 和问题排查。

---

# 17. Dependency Management

Fluvra 自己管理必要运行组件。

普通用户不应被要求：

- 安装 Python
- 安装 Homebrew
- 修改 PATH
- 手动安装 FFmpeg
- 手动安装 yt-dlp

依赖应由 Fluvra：

- 安装
- 验证
- 更新
- 回滚
- 检查完整性

依赖版本应该在 Settings 中可见，但不干扰日常使用。

---

# 18. Updates

Fluvra 至少区分以下更新渠道：

## Application

桌面应用本身。

---

## Download Core

负责网站解析和下载。

可以比应用本体更频繁更新。

---

## Media Engine

FFmpeg / FFprobe。

---

## JavaScript Runtime

用于部分网站解析。

---

## Integrations

网站特殊 Integration。

不同组件应允许独立升级。

---

# 19. Browser Integration

Browser Extension 属于 Phase 2。

目标浏览器：

- Chrome
- Edge
- Safari

首版 Extension 只需要：

- Download Video
- Download Audio
- Open in Fluvra

浏览器扩展不应该包含主要下载逻辑。

所有下载任务仍交给桌面应用。

---

# 20. Authentication

普通用户不应该首先看到 Cookie 文件。

标准交互：

> This media requires login.

主要操作：

> Sign In

后续支持：

- Browser Login
- Use Chrome Session
- Use Edge Session
- Use Firefox Session

高级模式可以支持：

- cookies.txt
- Custom Headers
- User-Agent
- Proxy

---

# 21. Smart Capture

Smart Capture 属于 Phase 3。

当普通 Resolver 无法解析媒体时：

Fluvra 可以提供：

> Open in Smart Capture

Smart Capture 用于检测：

- Direct Media
- HLS
- DASH
- Embedded Video
- Media Network Requests

Smart Capture 不应该成为普通下载流程的默认步骤。

优先级：

1. Standard Resolver
2. Integration Resolver
3. Smart Capture

---

# 22. Integrations

Fluvra 后期支持网站 Integration。

目标：

> 在不升级整个应用的情况下修复或者增加部分网站支持。

每个 Integration 应：

- 独立
- 可版本化
- 可更新
- 有明确权限范围
- 可以测试
- 可以禁用

未来可以建立：

> Community Integration Registry

并通过 GitHub 管理：

- Source
- Pull Requests
- Review
- Releases

---

# 23. Privacy

Fluvra 应尽可能 Local First。

默认情况下：

- 下载历史保存在本地
- 用户 URL 不上传
- Cookies 不上传
- 浏览历史不上传
- 媒体内容不经过 Fluvra 自有服务器

如果未来增加：

- Sync
- Crash Reporting
- Telemetry

必须：

- 明确说明
- 默认尊重隐私
- 提供关闭能力

---

# 24. Security

下载和 Integration 系统需要明确安全边界。

需要防止：

- 任意命令执行
- 任意文件覆盖
- Path Traversal
- 恶意文件名
- Integration 无限权限
- 未验证二进制更新
- 被篡改的依赖

所有核心依赖更新必须能够验证来源和完整性。

---

# 25. File Management

用户可以设置默认下载目录。

需要支持安全文件名处理：

- Invalid characters
- Duplicate filenames
- Excessive path length
- Unicode
- Reserved Windows names

发生重名时，默认不能静默覆盖已有文件。

例如：

`Video.mp4`

已有文件时：

`Video (1).mp4`

---

# 26. Cross-Platform Philosophy

Windows 和 macOS 共享：

- 产品逻辑
- 页面结构
- 核心组件
- Design Language

但允许针对平台做适配。

## macOS

可以使用：

- Menu Bar
- Command shortcuts
- Finder
- Native window behavior

## Windows

可以使用：

- System Tray
- Ctrl shortcuts
- Explorer
- Windows notifications

目标：

> 80% shared, 20% platform-native.

不强制两个平台逐像素完全一致。

---

# 27. MVP — v0.1

v0.1 的目标不是完成 Downie Clone。

目标是：

> 建立一个可靠、清晰、长期可扩展的下载核心产品。

必须支持：

### Platform

- Windows x64
- macOS ARM64

### Input

- Paste URL
- Drag URL

### Resolve

- Media information
- Thumbnail
- Duration
- Source
- Quality options
- Audio options
- Subtitle information
- Playlist detection

### Download

- Video
- Audio
- Playlist
- Subtitle
- Download queue
- Concurrent downloads
- Progress
- Speed
- ETA
- Cancel
- Retry

### Media Processing

- Merge
- Remux
- Audio extraction
- Subtitle handling

### Presets

- Best Quality
- High Compatibility
- Audio
- Archive
- Custom

### Library

- Download history
- Search
- Duplicate detection
- Open file
- Open folder
- Open source
- Download again

### Dependency

- Download Core management
- FFmpeg management
- FFprobe management
- JavaScript Runtime management

### Update

- App update
- Download Core update

### Error

- Error classification
- Human-readable errors

### Diagnostics

- Environment check
- Dependency versions
- Disk space
- Download folder status

---

# 28. Explicitly Out of Scope for v0.1

v0.1 不实现：

- Browser Extension
- Embedded Browser
- Smart Capture
- Integration Marketplace
- Community Plugins
- Cloud Sync
- Mobile Apps
- Remote Download
- NAS Integration
- Plex Integration
- Jellyfin Integration
- Advanced Automation
- Search Engine
- Top Downloads
- Social Features
- Accounts
- Fluvra Cloud
- DRM bypass

---

# 29. Phase 2 — Product

目标：

> 达到成熟商业下载器的日常使用体验。

计划功能：

- Chrome Extension
- Edge Extension
- Safari Extension
- Browser Login
- Browser Cookies
- URL Scheme
- Tray
- Menu Bar
- Advanced Presets
- Advanced Library
- Naming Rules
- Proxy
- Metadata
- Chapters
- Queue Reordering
- Priority
- Improved Error Recovery
- Diagnostic Export

---

# 30. Phase 3 — Smart

目标：

> 解决普通解析器无法覆盖的网站。

计划功能：

- Smart Capture
- Embedded Browser
- HLS Detection
- DASH Detection
- Direct Media Detection
- Login Browser
- Site Integrations
- Integration Registry
- Integration Update
- Custom Resolver API

---

# 31. Phase 4 — Platform

目标：

> 从下载应用发展成可扩展媒体获取平台。

可能功能：

- Automation
- Rules
- Plugin API
- Processing Pipelines
- WebDAV Sync
- Cross-device History
- NAS Workflow
- Plex Integration
- Jellyfin Integration
- Remote Queue
- Webhooks

Phase 4 的具体功能必须根据真实用户需求决定，不提前承诺全部实现。

---

# 32. Success Criteria

Fluvra 不以“支持多少参数”衡量成功。

核心指标应该是：

## Reliability

用户添加一个正常支持的 URL 后，可以可靠得到文件。

---

## Simplicity

普通用户无需理解 yt-dlp 或 FFmpeg。

---

## Recoverability

出现错误后，用户知道：

- 为什么失败
- 能不能恢复
- 下一步应该做什么

---

## Maintainability

网站发生变化时，可以通过：

- Download Core update
- Integration update

快速恢复，而不必频繁修改整个应用。

---

## Performance

应用本体保持轻量。

下载和媒体处理性能主要由：

- 网络
- yt-dlp
- FFmpeg

决定，而不是 GUI 成为瓶颈。

---

## Cross-Platform Quality

Windows 和 macOS 都应该被视为一等公民。

不能出现：

> macOS 是正式版，Windows 只是勉强能运行的移植版。

---

# 33. Product Decision Rules

以后出现新功能建议时，按照以下顺序判断。

### Question 1

它是否直接改善：

> 发现 → 下载 → 得到正确文件

如果是，优先级较高。

---

### Question 2

它是否解决真实、频繁出现的失败场景？

如果是，优先级较高。

---

### Question 3

它是否能通过现有功能组合完成？

如果可以，不急于增加新功能。

---

### Question 4

它是否让普通用户需要理解更多技术概念？

如果是，需要重新设计交互。

---

### Question 5

它是否明显增加长期维护成本？

如果是，需要有足够产品价值才能接受。

---

# 34. Non-Goals

Fluvra 不追求：

- 最多设置项
- 最多 yt-dlp 参数
- 最多导航页面
- 最复杂格式配置
- 最多视频编辑功能
- 支持所有极端网站
- 复制 Downie 的所有功能
- 完全复制某一个现有产品的 UI

Fluvra 追求：

> 用最少的用户决策，可靠得到正确的媒体文件。

---

# 35. Long-Term Product Identity

Fluvra 的最终身份应该是：

> 一个独立的媒体下载产品。

而不是：

> 一个带 GUI 的 yt-dlp。

因此，未来 README 的主标题应该描述 Fluvra 自己。

例如：

> **A modern media downloader for Windows and macOS.**

而不是：

> **A GUI frontend for yt-dlp.**

yt-dlp、FFmpeg 和其他依赖可以在技术说明与 Credits 中明确注明。

它们负责提供能力。

Fluvra 负责提供产品。

---

# 36. Current Product Status

当前阶段：

> Product Definition

尚未进入：

- Architecture Freeze
- Design Freeze
- Implementation

下一份核心文档：

`ARCHITECTURE.md`

它需要回答：

- Core 模块如何划分
- Resolver 如何抽象
- Download Task 数据模型
- Process 生命周期
- FFmpeg Pipeline
- Binary Manager
- Update Manager
- SQLite Schema
- Frontend / Backend IPC
- Integration 安全模型
- Windows/macOS 平台边界

在 `ARCHITECTURE.md` 完成前，不应该开始大规模实现业务代码。
