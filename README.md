这是一个为你量身定制的 `README.md` 文档草案。结合了你提供的 Docker 配置、`config.yaml` 业务逻辑、Telegram Bot 命令列表，以及我们在历史讨论中沉淀的核心技术点（如 115 API 加密处理、QPS 风控、缓存优化和事件监控）。

你可以直接复制这段 Markdown 代码到你的项目仓库中。

---

# PS2115 - 影视资源自动化聚合与转存系统 🎬

PS2115 是一个专为 115 网盘用户打造的影视资源自动化管理工具。它集成了强大的 Telegram Bot 交互、多源资源聚合搜索、全自动订阅追剧以及 115 网盘的深度本地化管理（包含增量同步与事件监控）。

## ✨ 核心特性

* **🤖 极致的 Telegram Bot 交互**：采用高屏效比设计，防误触 URL 按钮排版，实现极简的搜索、转存、订阅体验。
* **🔍 双引擎资源聚合**：
* `CloudSaver`：直接抓取各大 Telegram 公开资源频道，深度接入自动订阅系统。
* `Pansou`：独立聚合搜索引擎 API，提供全网泛搜（仅用于直接转存/离线，无缝补充资源库）。


* **⚡ 115 深度整合与防风控**：
* 内置 115 生活事件增量监控，支持特定 CID 的精准捕获与目录树血缘校验。
* 适配 115 最新 Pro API 加密与解密逻辑（解决 `decode fail!` 报错）。
* 严格且安全的 QPS 控制（动态节流至 0.5 - 1.0），保护账号免受风控。


* **🚀 高性能架构**：后台 Cron 定时拉取与前端 API 手动触发双引擎策略；针对播放器设计的 UA 前缀智能缓存机制（大幅减少 302 重定向请求频率）。

---

## 📦 快速部署 (Docker)

推荐使用 Docker Compose 进行一键部署。

### 1. 编写 `docker-compose.yml`

```yaml
version: '3.8'

services:
  ps2115:
    image: lisylva/ps2115:latest # 或者指定版本号，如 v1.5.9
    container_name: ps2115
    restart: unless-stopped
    ports:
      - "9797:8080" # Web UI / API 端口
      - "8099:8099" # 内部通信/其他服务端口
    volumes:
      - ./data:/app/data
      - ./strm:/app/strm
      - ./config.yaml:/app/config.yaml
    environment:
      - TZ=Asia/Shanghai

```

### 2. 准备配置文件

在映射的目录下创建 `config.yaml`，参考以下配置：

```yaml
# 基础服务配置
server:
  username: admin
  password: admin123
  jwt_secret: ps2115-secret-key-change-me

# CloudSaver 搜索引擎
cloudsaver:
  channels:
    - id: "pindaoh"
      alias: "影视资源"

# Pansou 聚合搜索 API：独立搜索引擎（用于临时搜索及转存，不接入订阅）
pansou:
  url: "https://so.252035.xyz/"

```

### 3. 启动服务

```bash
docker-compose up -d

```

---

## 📱 Telegram Bot 交互指南

绑定并启动 Bot 后，你可以通过以下命令在 Telegram 中直接管理你的媒体库：

| 命令 | 参数 | 说明 |
| --- | --- | --- |
| `/search` | `<关键词>` | 搜索 115 资源（走 Cloudsaver 引擎，支持订阅联动） |
| `/ps` | `<关键词>` | 盘搜搜索（走 Pansou 引擎，支持直接转存/离线下载） |
| `/sub add` | `<关键词> | <目录>` | 添加新的追剧订阅规则 |
| `/sub list` | 无 | 查看当前所有订阅任务的状态 |
| `/sub del` | `<ID>` | 删除指定的订阅 |
| `/sub pause` | `<ID>` | 暂停指定的订阅任务 |
| `/sub resume` | `<ID>` | 恢复已暂停的订阅任务 |
| `/sub check` | `<ID>` | 立即触发一次订阅检查，拉取最新资源 |
| `/status` | 无 | 查看系统运行状态、资源占用及 QPS 监控 |
| `/cookie` | `<cookie>` | 快速更新或重置 115 网盘的 Cookie 凭证 |
| `/help` | 无 | 显示使用帮助与命令列表 |

---

## 🛠️ 架构与高阶功能说明

### 115 网盘事件监控与 CID 独立控制

本项目并非单纯的定时全量扫盘。系统实现了**三层独立控制机制**：

* 生活事件增量同步（精准抓取变化）。
* 间隔巡检 CID 列表。
* 全树巡检 CID 配置。
数据持久化采用 SQLite 的 JSON 序列化存储，确保防风控限速下万级文件的平滑扫盘，即便中断也能自我修复对齐。

### 搜索引擎差异说明

* **搜索资源 (Cloudsaver)**：结果会作用于订阅管理系统。当配置的 TG 频道发布新资源时，系统会自动匹配您的 `/sub` 规则完成自动化转存。
* **盘搜 (Pansou)**：作为资源补充的独立入口，查到的资源仅供即时 `[📥 转存]` 或 `[⬇ 离线]` 使用，**不会**污染或触发你的日常自动化订阅流。
