# LogFilter

[![Build and Release](https://github.com/thunder951413/log_filter/actions/workflows/build.yml/badge.svg)](https://github.com/thunder951413/log_filter/actions/workflows/build.yml)
[![GitHub release](https://img.shields.io/github/v/release/thunder951413/log_filter)](https://github.com/thunder951413/log_filter/releases)
[![Platforms](https://img.shields.io/badge/platforms-macOS%20%7C%20Windows%20%7C%20Linux-blue)](https://github.com/thunder951413/log_filter/releases)

LogFilter 是一个面向本地日志分析场景的桌面工具，使用 Python + Dash 提供分析能力，并通过 Electron 打包为 macOS / Windows / Linux 桌面应用。

它适合处理体积较大的日志文件、复用关键字过滤规则、按问题场景管理配置，并把原本依赖浏览器访问的分析工具整理成更容易分发和使用的桌面程序。

### 如果要追求日志过滤性能（超大日志类）需要配置 rg 等工具并放到系统 path 环境。
RG：https://github.com/burntsushi/ripgrep

## 为什么用 LogFilter

- 快速过滤日志中的关键信息，减少人工翻找
- 通过配置文件复用常见分析规则
- 支持配置组，适合 CI、播放链路、业务模块等不同场景
- 支持桌面端拖拽导入日志文件
- 支持 GitHub Actions 自动构建和发布多平台安装包

## 主要价值

LogFilter 的核心价值是把"人工翻日志"变成"规则化过滤、流程化查看、可复用沉淀、可 AI 辅助分析"的本地日志排查工作台。

- **提升日志排查效率**：通过保留关键字、屏蔽关键字、配置组和临时关键字快速缩小日志范围，避免在大日志中逐行查找。
- **沉淀可复用分析规则**：把常见问题场景保存为配置文件和配置文件组，遇到类似问题时可以直接复用。
- **适合大日志本地分析**：使用异步过滤、临时文件、行索引和滚动窗口分片加载，降低大文件一次性渲染带来的性能压力。
- **支持问题对比定位**：对两份日志使用同一规则过滤后进行左右对比，便于比较正常/异常日志、不同设备日志或不同版本日志。
- **让日志更容易理解**：通过关键字注释、流程视图和序列流程，把零散日志还原成更接近业务流程的视图。
- **降低工具使用门槛**：提供可视化页面和桌面应用形态，减少对命令行过滤工具的依赖。
- **连接 AI 辅助分析**：可选择关键日志片段并携带配置组、日志文件、关联配置文件和选中行数等上下文交给 AI 分析，辅助定位原因并沉淀经验。

## 快速开始

### 直接下载桌面版

前往 [Releases](https://github.com/thunder951413/log_filter/releases) 下载对应平台产物：

- macOS：`LogFilter-macOS.zip` （因为非开发者账号，所以需要本地用 xattr -rc /Application/LogFilter 来修复权限）
- Windows：`LogFilter-Windows.zip`
- Linux：`LogFilter-Linux.tar.gz`

解压后即可运行。

## 核心能力

### 日志过滤

- 从 `logs/` 目录选择 `.txt` / `.log` / `.text` 日志文件进行分析
- 支持"保留关键字"和"屏蔽关键字"两类规则，适合快速收敛大日志中的有效片段
- 支持临时关键字，不需要写入长期配置即可临时追加保留或屏蔽条件
- 支持按配置文件组批量加载规则，例如 `COMMON`、`TCL`、`ROKU` 等问题场景
- 支持过滤结果、源文件、高亮显示、注释、流程视图五种展示模式
- 支持关键字高亮、全局搜索上一个/下一个、行号跳转、快速回到顶部/底部
- 支持选择日志行并发送到 AI 分析入口，便于围绕具体错误片段继续定位
- 后台异步过滤，前端通过进度条轮询任务状态
- 过滤后端支持自动选择 `rg` (ripgrep) > `grep` > `findstr` > PowerShell > Python 流式回退

### 大日志处理

- 过滤任务在后台线程中异步执行，页面通过进度条轮询任务状态
- 过滤结果先写入 `temp/` 临时文件，并生成行偏移索引，避免一次性把大文件全部渲染到页面
- 结果视图使用滚动窗口分片加载（虚拟滚动），适合查看较大的过滤结果
- 自动检测文件编码，并优先使用字节级匹配降低逐行解码开销
- 支持过滤后高亮缓存（`HighlightCache`，LRU + SHA1 键），减少重复渲染开销

### 日志对比

- 支持选择两份日志文件，使用同一组过滤规则先过滤再对比
- 支持按配置文件组加载对比规则
- 支持设置"忽略行首 N 个字符"，用于跳过时间戳、线程号等易变前缀
- 提供左右分栏的差异视图和同步滚动开关（`compare_sync.js`），便于对比两次运行或两台设备的差异

### 桌面打包

- Python 后端负责 Dash 页面与日志分析逻辑
- Electron 主进程负责启动后端、创建窗口和桌面能力
- PyInstaller + electron-builder 负责生成桌面发行包

## 桌面版说明

桌面版采用 Electron + Python 双进程架构：

- Electron 主进程启动本地后端服务
- Python 后端提供 Web 界面和分析能力
- 桌面窗口在后端就绪后再加载页面

首次启动桌面版时，后端初始化可能需要一段时间。当前版本已增加启动等待页和失败提示页，避免后端未就绪时直接出现白屏。

## UI 布局

应用为 Dash 单页应用 (SPA)，使用 `dash-bootstrap-components` 构建响应式布局，包含以下主要 Tab 页面：

| Tab | 功能 | 关键组件 |
|-----|------|---------|
| 日志管理 | 上传/删除/选择日志文件 | 文件列表 + 拖拽上传区 + 外部程序配置 |
| 配置管理 | 编辑关键字分类、配置文件、配置组 | 关键字编辑器 + 分类选择器 + JSON 预览 |
| 日志过滤 | 核心过滤功能 | 文件选择器 + 配置组选择器 + 关键字输入 + 过滤按钮 + 进度条 |
| 日志对比 | 双栏日志对比 | 左右文件选择器 + 配置组选择器 + 差异视图 |

过滤结果包含 5 种展示模式（子 Tabs 切换）：过滤结果、源文件、高亮显示、注释、流程视图。

## 技术栈

### 后端 (Python)
| 技术 | 用途 |
|------|------|
| Python 3.10+ | 主开发语言 |
| Dash 2.x | Web 框架（Flask + React） |
| dash-bootstrap-components | UI 组件库 |
| Plotly | 数据可视化 |
| Pandas | 数据处理 |
| OpenAI >=1.0.0 | AI 分析 (LLM 调用) |
| PyInstaller | Python 打包为单文件可执行 |

### 前端 (浏览器)
| 技术 | 用途 |
|------|------|
| Dash 自动生成 | 核心 UI |
| 原生 JavaScript (vanilla JS) | 所有 assets/*.js 为手写原生 JS |
| CSS3 | 现代化 UI（毛玻璃效果、动画） |
| Fetch API | 与后端 HTTP 通信 |
| SSE (Server-Sent Events) | AI 聊天流式响应 |
| ResizeObserver / MutationObserver | DOM 变化监听 |
| localStorage | 会话持久化 |

### 桌面壳 (Electron)
| 技术 | 用途 |
|------|------|
| Electron 39.x | 桌面应用壳 |
| electron-builder 24.x | 多平台打包 |
| electron-updater 6.x | 自动更新 |
| electron-log 5.x | 日志系统 |

### 构建与 CI/CD
| 技术 | 用途 |
|------|------|
| npm | Node.js 包管理 |
| pip | Python 包管理 |
| PyInstaller | Python → 单文件可执行 |
| electron-builder | Electron → 桌面安装包 |
| GitHub Actions | 自动构建/发布 |

### AI 集成
| 技术 | 用途 |
|------|------|
| freecode-cli | 本地 LLM CLI 工具 |
| OpenAI SDK | LLM 调用（兼容 OpenAI API） |
| Agentic Loop | LLM 自主工具调用（4 个工具） |

## License

如需许可证信息，请查看仓库根目录中的 LICENSE 文件。
