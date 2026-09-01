# ARCHITECTURE.md

> Version: 0.1.0  
> Status: Architecture Baseline  
> Product: Fluvra  
> Last updated: 2026-09-01

---

# 1. Purpose

本文定义 Fluvra v0.1 的技术架构基线，承接 `PRODUCT.md`，用于冻结主技术路线、核心模块边界、进程/组件生命周期、数据存储、跨平台边界、安全边界与系统落盘规则。

实现细节可以演进，但不得破坏本文定义的核心边界。

---

# 2. Architecture Decisions

Fluvra v0.1 正式采用：

```text
Frontend              React + TypeScript + Vite
Desktop Runtime       Tauri 2
Core                   Rust
Database               SQLite
Download Resolver      yt-dlp Adapter
Media Engine           FFmpeg + FFprobe
JavaScript Runtime     Fluvra-managed Deno
Initial Platforms      Windows 11 x86-64 / macOS Apple Silicon
```

Swift 仅允许出现在 macOS platform adapter 中，不作为共享 Core 或主 UI 技术栈。

---

# 3. Core Principles

1. React 只负责 UI、交互和展示状态，不直接执行 yt-dlp、FFmpeg 或系统命令。
2. Rust Core 是下载任务、队列、进程、组件、媒体处理、持久化和错误分类的权威来源。
3. Windows 与 macOS 共享 Domain、Resolver、Queue、SQLite、Media Pipeline、React UI 与 IPC contract；平台差异集中在 platform layer。
4. Fluvra 自己管理 yt-dlp、FFmpeg、FFprobe、Deno，不要求用户安装 Python、Homebrew、Chocolatey、修改 PATH 或手动维护依赖。
5. 下载任务是持久化 Domain Object，不是一次性的 shell command。
6. Frontend、Integration、External Binary、Filesystem 和 Credentials 之间必须有明确安全边界。
7. 所有 Fluvra 创建的系统资源都必须有明确所有权与 cleanup lifecycle。

---

# 4. Clean System Footprint

这是硬性架构规则。

> **Fluvra owns everything it creates, knows where everything it creates lives, and can clean everything it creates.**

禁止：

- 修改系统 PATH
- 全局安装 yt-dlp / FFmpeg / FFprobe / Deno
- 自动调用 Homebrew / winget / Chocolatey 安装运行依赖
- 在用户 Home 根目录创建 `.fluvra`
- 在 Downloads 中创建隐藏运行目录
- 在应用安装目录旁写 database/cache/logs
- 向 System32、`/usr/local/bin`、`/opt/homebrew` 等系统位置复制运行文件
- 无理由创建 Service / LaunchDaemon / Scheduled Task
- 无理由写系统级 Registry

所有路径必须通过统一 `PathService` 获取，业务代码不得硬编码系统目录。

---

# 5. Platform Data Layout

## macOS

Application：

```text
/Applications/Fluvra.app
```

Persistent data：

```text
~/Library/Application Support/Fluvra/
├── data/
│   └── fluvra.db
├── components/
├── integrations/
└── state/
```

Cache：

```text
~/Library/Caches/Fluvra/
```

Logs：

```text
~/Library/Logs/Fluvra/
```

Temporary working files：

```text
$TMPDIR/Fluvra/
```

## Windows

Fluvra-owned writable data：

```text
%LOCALAPPDATA%\Fluvra\
├── data\
│   └── fluvra.db
├── components\
├── integrations\
├── cache\
├── logs\
└── state\
```

Temporary working files：

```text
%TEMP%\Fluvra\
```

不得创建 `C:\Fluvra`、`C:\ffmpeg`、`%USERPROFILE%\.fluvra` 等散落目录。

用户下载的视频、音频、字幕属于 User-owned files，Reset 或 Uninstall 不得自动删除。

---

# 6. High-Level Architecture

```text
┌─────────────────────────────────────────────┐
│              React / TypeScript             │
│ Home · Downloads · Library · Settings       │
└──────────────────────┬──────────────────────┘
                       │ Typed IPC
                       ▼
┌─────────────────────────────────────────────┐
│                 Tauri 2                     │
│ Commands · Events · Window · Permissions    │
└──────────────────────┬──────────────────────┘
                       ▼
┌─────────────────────────────────────────────┐
│                 Rust Core                   │
│ Resolver · Download · Queue · Task          │
│ Process · Media · Library · Components      │
│ Updates · Diagnostics · Security · Storage  │
└───────────┬───────────────┬─────────────────┘
            │               │
            ▼               ▼
┌───────────────────┐   ┌────────────────────┐
│ External Runtime  │   │ Platform Adapters  │
│ yt-dlp            │   │ macOS / Windows    │
│ FFmpeg / FFprobe  │   │ Native APIs only   │
│ Deno              │   │                    │
└───────────────────┘   └────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────┐
│                   SQLite                    │
│ Tasks · Library · Settings · Components     │
└─────────────────────────────────────────────┘
```

依赖方向：

```text
Frontend
   ↓
Application Commands
   ↓
Domain Services
   ↓
Infrastructure Adapters
   ↓
OS / SQLite / yt-dlp / FFmpeg / Deno
```

---

# 7. Repository Structure

```text
Fluvra/
├── src/
│   ├── app/
│   ├── features/
│   │   ├── home/
│   │   ├── downloads/
│   │   ├── library/
│   │   └── settings/
│   ├── shared/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── ipc/
│   │   ├── types/
│   │   └── utils/
│   └── styles/
├── src-tauri/
│   ├── src/
│   │   ├── commands/
│   │   ├── domain/
│   │   │   ├── resolver/
│   │   │   ├── download/
│   │   │   ├── queue/
│   │   │   ├── media/
│   │   │   ├── library/
│   │   │   └── errors/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   │   ├── process/
│   │   │   ├── sqlite/
│   │   │   ├── components/
│   │   │   └── updater/
│   │   ├── platform/
│   │   │   ├── macos/
│   │   │   └── windows/
│   │   └── diagnostics/
│   ├── migrations/
│   ├── capabilities/
│   └── Cargo.toml
├── scripts/
├── docs/
├── PRODUCT.md
├── ARCHITECTURE.md
└── README.md
```

目录名可以调整，但模块边界不能被压平成巨大 command 文件。

---

# 8. IPC Boundary

Tauri Commands 用于请求/响应，例如：

```text
resolve_media(url)
create_download(request)
cancel_task(task_id)
retry_task(task_id)
list_tasks()
list_library(query)
run_diagnostics()
check_component_updates()
```

Tauri Events 用于持续状态变化，例如：

```text
task://created
task://updated
task://progress
task://stage-changed
task://completed
task://failed
component://updated
```

规则：

- IPC DTO 与 Domain Model 分离
- 不向 Frontend 暴露原始 yt-dlp JSON 作为产品 API
- 不向 Frontend 暴露任意 shell execution
- 不让 Frontend 拼接 FFmpeg / yt-dlp command line

---

# 9. Resolver Architecture

```text
URL
 ↓
URL Validator / Normalizer
 ↓
Resolver Service
 ↓
Resolver Adapter
 ↓
yt-dlp
 ↓
Normalize
 ↓
ResolvedMedia
```

v0.1：

```text
Resolver
└── YtDlpResolver
```

未来：

```text
Resolver Chain
├── StandardResolver
├── IntegrationResolver
└── SmartCaptureResolver
```

上层业务不得直接依赖 yt-dlp format JSON 结构。

---

# 10. Core Domain Models

## ResolvedMedia

建议至少包含：

```text
source_url
canonical_url
source
media_id
media_type
title
creator
description
thumbnail
duration
upload_date
formats
audio_tracks
subtitles
chapters
playlist
```

## MediaFormat

建议字段：

```text
source_format_id
kind
container
video_codec
audio_codec
width
height
fps
bitrate
sample_rate
channels
filesize
filesize_approx
hdr
language
```

UI 不把 yt-dlp format ID 当成主要产品交互。

---

# 11. Preset and Download Plan

Preset 是语义策略，不是 yt-dlp 参数字符串。

```text
Best Quality
High Compatibility
Audio
Archive
Custom
```

转换：

```text
Preset + ResolvedMedia + User Options
                ↓
        Download Planner
                ↓
          DownloadPlan
```

`DownloadPlan` 建议包含：

```text
media
selected_video
selected_audio
selected_subtitles
destination
temp_directory
filename
output_container
download_options
processing_steps
```

---

# 12. Download Task Model

Task 使用稳定 UUID。

```text
DownloadTask
├── id
├── source_url
├── canonical_url
├── media_id
├── title
├── preset
├── plan
├── status
├── stage
├── progress
├── downloaded_bytes
├── total_bytes
├── speed
├── eta
├── output_path
├── error
├── retry_count
├── created_at
├── updated_at
└── completed_at
```

Task 与 Process 不是一一等价关系；一个 Task 可以先后拥有 metadata、download、merge、validation 等多个 process。

---

# 13. Task State Machine

一级状态：

```text
Resolving
Pending
Waiting
Downloading
Processing
Completed
Failed
Cancelled
```

Status 表示生命周期，Stage 表示当前实际步骤，例如：

```text
Resolving Metadata
Downloading Video
Downloading Audio
Downloading Subtitles
Merging
Remuxing
Converting
Embedding Subtitles
Embedding Metadata
Validating
Finalizing
```

合法状态转换由 Rust Core 控制。

---

# 14. Queue Architecture

Queue Service 统一控制：

- FIFO
- Configurable concurrency
- Retry
- Cancel

Phase 2 增加：

- Reorder
- Priority

应用启动时从数据库恢复非终态 Task；不得假设旧 process 仍存在。

---

# 15. Process Manager

所有 external process 必须由 Rust `ProcessManager` 创建和持有。

负责：

- spawn
- stdout/stderr collection
- progress parsing
- cancellation
- graceful termination
- forced termination fallback
- exit code collection
- child cleanup

禁止业务模块随意 `Command::spawn()`。

Windows 必须使用适当的 child-process ownership 机制，避免主程序退出后遗留孤儿进程；macOS 同样必须保证所有 child process 可追踪、可终止。

---

# 16. Command Construction

禁止构造 shell command string：

```text
"yt-dlp " + user_input
```

必须始终使用：

```text
Executable + Argument Array
```

用户 URL、路径、文件名、headers 等均作为独立参数传递，不通过 shell interpolation 执行。

---

# 17. yt-dlp Adapter

Fluvra 不把 yt-dlp 当成 Product API。

建议拆分：

```text
YtDlpResolver
YtDlpDownloader
YtDlpProgressParser
YtDlpErrorClassifier
```

优先使用机器可读输出，不依赖面向人类的 console 文案作为稳定协议。

Fluvra 应主动避免读取用户全局 yt-dlp 配置，以保证运行结果可预测。

高级 Custom Arguments 必须经过安全策略；Power User Mode 不等于任意命令执行。

---

# 18. JavaScript Runtime

Fluvra v0.1 管理 Deno，并显式把 runtime path 交给 yt-dlp，不依赖系统 PATH。

```text
Component Manager
      ↓
Managed Deno
      ↓
Explicit runtime path
      ↓
yt-dlp
```

---

# 19. Media Pipeline

```text
Downloaded Assets
      ↓
Media Plan
      ↓
Merge / Remux / Convert / Embed
      ↓
FFmpeg
      ↓
FFprobe Validation
      ↓
Final File
```

优先复用 yt-dlp 已成熟的 post-processing；只有需要明确可控的产品步骤时，才直接调用 FFmpeg，避免重复实现成熟能力。

最终文件通过 FFprobe 验证并提取 Library metadata。

---

# 20. Component Manager

管理：

```text
yt-dlp
FFmpeg
FFprobe
Deno
```

必须支持：

- platform / architecture selection
- version discovery
- download
- integrity verification
- extraction
- executable permission
- health check
- activation
- rollback
- old-version cleanup

组件采用版本目录，不直接覆盖正在使用的文件：

```text
components/
├── yt-dlp/
│   ├── <version>/
│   └── active.json
├── ffmpeg/
│   ├── <version>/
│   └── active.json
├── ffprobe/
│   ├── <version>/
│   └── active.json
└── deno/
    ├── <version>/
    └── active.json
```

更新流程：

```text
Download
  ↓
Verify
  ↓
Extract new version
  ↓
Health Check
  ↓
Atomic Activation
  ↓
Retain rollback version
  ↓
Cleanup obsolete versions
```

失败不得破坏当前可用版本。

---

# 21. Bootstrap Components

安装包可以携带首次启动所需的 bootstrap component versions，使普通用户无需先配置系统依赖。

运行后，后续组件版本统一由 Component Manager 管理在 Fluvra-owned data 目录。

App Update 与 Component Update 必须解耦。

---

# 22. Task Temporary Files

每个 Task 使用隔离工作目录：

```text
temp/
└── <task-uuid>/
    ├── downloads/
    ├── processing/
    └── metadata/
```

成功后清理不再需要的中间文件；取消后尽可能清理；失败时根据恢复策略保留必要 partials；超期 partials 由 cleanup policy 清理。

不得静默删除仍可用于 Resume 的数据。

---

# 23. Filename Safety

统一 Filename Service 处理：

- invalid characters
- Windows reserved names
- Unicode
- trailing dots/spaces
- excessive path length
- duplicate names
- path traversal

远程 metadata 绝不能直接成为未经处理的 filesystem path。

重名默认：

```text
Video.mp4
Video (1).mp4
Video (2).mp4
```

不静默覆盖已有文件。

---

# 24. SQLite Architecture

SQLite 是 v0.1 唯一本地业务数据库。

原则：

- WAL mode
- foreign keys enabled
- explicit migrations
- transactional writes
- schema versioning

Frontend 不直接打开 SQLite 文件。

初始建议表：

```text
schema_migrations
settings
components
download_tasks
task_events
library_items
library_files
```

高频 progress 主要走 memory/event stream；status/stage transition 与 terminal state 立即持久化，避免 SQLite 成为进度瓶颈。

---

# 25. App Lifecycle and Recovery

v0.1 不引入独立后台 daemon。

关闭应用后，不保证下载继续运行。

正常关闭：

```text
Stop accepting new tasks
      ↓
Signal active tasks
      ↓
Persist latest state
      ↓
Terminate owned processes
      ↓
Flush DB/logs
      ↓
Exit
```

异常退出后，下次启动：

```text
Open DB
  ↓
Run migrations
  ↓
Find non-terminal tasks
  ↓
Inspect working files
  ↓
Recover / Retry / Mark recoverable failure
```

---

# 26. Error Model

统一转换为：

```text
FluvraError
├── code
├── category
├── title
├── message
├── recoverable
├── suggested_action
└── technical_detail
```

至少覆盖：

```text
NETWORK_ERROR
TIMEOUT
AUTH_REQUIRED
LOGIN_EXPIRED
UNSUPPORTED_URL
MEDIA_UNAVAILABLE
GEO_RESTRICTED
RATE_LIMITED
CORE_OUTDATED
DISK_FULL
FILE_PERMISSION
POSTPROCESS_FAILED
DEPENDENCY_MISSING
CANCELLED
UNKNOWN_ERROR
```

Frontend 主要展示 product-level error，technical detail 仅用于 Diagnostics / Advanced mode。

---

# 27. Logging and Credentials

日志统一由 Rust 管理，必须：

- rotation
- retention policy
- 标准 Fluvra log directory
- task_id correlation
- component version 信息

必须过滤或脱敏：Cookies、Authorization headers、access tokens、secrets、private headers。

未来凭证通过 `CredentialProvider` 抽象，长期秘密应使用平台安全存储，不以明文随意写入普通 settings JSON。

---

# 28. Security Boundary

必须防止：

- arbitrary command execution
- shell injection
- path traversal
- arbitrary file overwrite
- unverified binary replacement
- unrestricted Integration filesystem access
- credential leakage
- frontend unrestricted shell access

Tauri capability 使用最小权限原则。Frontend 不获得泛化任意 shell spawn 权限；External binary 启动集中在 Rust Core。

---

# 29. Integrations

Integrations 不属于 v0.1，但架构必须为未来保留：

- independently versioned
- capability scoped
- disableable
- testable
- updatable

Integration 不得默认获得任意 process、filesystem、database、credential、component replacement 权限。

Integration 输出重新进入 Fluvra Domain Model。

---

# 30. Platform Layer

平台代码集中：

```text
platform/
├── macos/
└── windows/
```

Domain Service 不应散落大量平台条件逻辑。

## macOS

优先使用 Rust/Tauri 已稳定支持的能力。只有 Apple 原生 API 明显更合适时，才增加 Swift / Objective-C bridge，例如 Safari Extension、Finder integration、Login Items、Keychain、AppKit 特殊行为。

Swift 不承担共享业务 Core。

## Windows

Windows adapter 负责 Explorer integration、protocol/file registration、process ownership、installer/uninstaller、Win32/WinRT 能力。

业务 Domain 不直接依赖 Win32 types。

---

# 31. Update Architecture

App Update 与 Component Update 分开：

```text
Application Update
≠ yt-dlp Update
≠ FFmpeg Update
≠ Deno Update
≠ Integration Update
```

桌面 App Update 使用 Tauri 2 Updater。

Component Update 由 Fluvra 自己控制 manifest，至少描述：

```text
component
version
platform
architecture
url
hash
size
release_date
```

不得把 `yt-dlp -U` 作为主要更新模型。

任何更新都遵守：旧版本保持可用 → 下载新版本 → 验证 → Health Check → 激活。

---

# 32. Diagnostics

至少检查：

```text
Application Version
OS / Architecture
Database Version
Download Core Version
FFmpeg Version
FFprobe Version
Deno Version
Component Health
Download Directory
Write Permission
Temporary Directory
Free Disk Space
Update State
```

诊断报告应可导出、可读并自动脱敏。

---

# 33. Packaging

初始 release targets：

```text
Windows 11 x86-64
macOS Apple Silicon / aarch64
```

CI 在对应平台构建对应产物。

macOS Release 需要 signing + notarization；Windows Release 需要 proper installer，并在 release infrastructure ready 后进行 code signing。

第三方组件必须维护 component name、source、version、license、notices 与 build provenance；尤其 FFmpeg 二进制发布前需要按实际 build configuration 做 license review。

---

# 34. Test Strategy

测试保护关键行为，不追求形式化高覆盖率。

Unit 优先：

- filename sanitization
- preset → download plan
- task state transitions
- error classification
- component version selection

Integration 优先：

- yt-dlp machine output → ResolvedMedia
- process cancellation
- SQLite migrations
- component activation / rollback

Smoke 优先：

- app startup
- resolver basic path
- controlled sample download
- FFmpeg invocation

不为简单 getter、文案和低风险代码堆积测试。

---

# 35. Performance Rules

- progress events 需要 throttling
- 不把所有 stdout line 发给 React
- 不把高频 progress 全部写 SQLite
- thumbnail/image cache 有明确限制
- 大文件不通过 IPC 传 binary payload
- media files 始终通过 filesystem path 管理

---

# 36. Explicit v0.1 Non-Goals

v0.1 不引入：

- Electron
- Flutter
- SwiftUI + WinUI 双 UI
- local HTTP server
- GraphQL / WebSocket server
- Redis / message broker
- background daemon/service
- plugin marketplace
- embedded browser
- Smart Capture
- mobile architecture
- cloud backend
- custom replacement for yt-dlp
- custom replacement for FFmpeg

---

# 37. Implementation Phases

## P0 — Foundation

Tauri 2、React + TypeScript + Vite、Rust skeleton、IPC、Path Service、SQLite migrations、logging。

## P1 — Component Runtime

Component Manager、yt-dlp、FFmpeg、FFprobe、Deno、health checks、version selection。

## P2 — Resolver

URL validation、YtDlpResolver、ResolvedMedia、format normalization、basic error classifier。

## P3 — Download

DownloadPlan、DownloadTask、state machine、Queue、Process Manager、progress、cancel、retry。

## P4 — Media

merge、remux、audio extraction、subtitle processing、FFprobe validation。

## P5 — Product UI

Home、Downloads、Library、Settings。

## P6 — Reliability

crash recovery、partial file policy、duplicate detection、diagnostics、cleanup policies。

## P7 — Updates

Application Updater、Component Updater、verification、activation、rollback。

---

# 38. Architecture Invariants

以下为 v0.1 不可悄悄违反的规则：

1. React 不直接执行 external binaries。
2. Rust Core 是下载任务状态的权威来源。
3. yt-dlp / FFmpeg / FFprobe / Deno 由 Fluvra 管理，不要求全局安装。
4. 不修改系统 PATH。
5. 所有 Fluvra-owned writable data 必须位于标准应用目录或明确临时目录。
6. User-owned media 与 app-owned data 永远区分。
7. External Process 必须由 Process Manager 管理。
8. Shell command 不通过字符串拼接执行。
9. SQLite 只由 Rust 业务层直接访问。
10. App Update 与 Component Update 分离。
11. Windows 和 macOS 都是一等平台。
12. 平台特有代码集中在 platform layer。
13. v0.1 不引入 daemon、local HTTP server 或 microservice。
14. Power User 功能不能绕过基础安全边界。
15. Fluvra 创建的系统资源必须存在对应 cleanup lifecycle。

违反这些规则前，必须先修改架构文档或新增明确 ADR。

---

# 39. Open Decisions

以下问题留到实现阶段局部决定：

- React 状态管理库
- SQLite Rust crate
- IPC type generation 方案
- Windows installer 最终格式
- Component manifest 托管形式
- 自动更新默认策略
- 日志 retention 天数
- 默认并发数量
- rollback versions 保留数量
- Advanced yt-dlp args allowlist

这些不影响当前整体架构方向。

---

# 40. Architecture Status

当前冻结的核心方向：

> **Tauri 2 + Rust Core + React/TypeScript + SQLite + managed yt-dlp/FFmpeg/FFprobe/Deno**

以及：

> **Shared product/core first, platform-specific behavior only at explicit boundaries.**

和：

> **Clean System Footprint: Fluvra 不在 Windows 或 macOS 中散落自身文件和运行依赖。**

完成本文后，可以开始 P0 Foundation Implementation。

---

# 41. Technical References

- Tauri 2 — Embedding External Binaries: https://v2.tauri.app/develop/sidecar/
- Tauri 2 — Updater: https://v2.tauri.app/plugin/updater/
- yt-dlp: https://github.com/yt-dlp/yt-dlp
- yt-dlp EJS / External JavaScript Runtime guidance: https://github.com/yt-dlp/yt-dlp/wiki/EJS
- FFmpeg: https://ffmpeg.org/
- Deno: https://deno.com/
