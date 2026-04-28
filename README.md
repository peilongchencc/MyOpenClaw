# MyOpenClaw

开始养虾🦐
- [MyOpenClaw](#myopenclaw)
  - [笔者电脑配置](#笔者电脑配置)
  - [新手上手建议](#新手上手建议)
  - [打开可视化 Dashboard（浏览器访问 http://127.0.0.1:18789）](#打开可视化-dashboard浏览器访问-http12700118789)
    - [聊天](#聊天)
    - [控制（运维监控区）](#控制运维监控区)
    - [代理（AI 智能体区）](#代理ai-智能体区)
    - [设置](#设置)
  - [设定IDENTITY.md](#设定identitymd)
  - [心跳监测机制:](#心跳监测机制)
  - [更新openclaw](#更新openclaw)
  - [openclaw gateway status--查看openclaw网关状态](#openclaw-gateway-status--查看openclaw网关状态)
  - [openclaw doctor --repair](#openclaw-doctor---repair)
  - [OpenClaw Gateway 使用 NVM Node 的修复指南](#openclaw-gateway-使用-nvm-node-的修复指南)
    - [错误信息示例:](#错误信息示例)
    - [背景：为什么会出现这个问题？](#背景为什么会出现这个问题)
    - [修复方案:](#修复方案)
    - [验证修复效果](#验证修复效果)

## 笔者电脑配置

| 配置项 | 版本信息 |
| --- | --- |
| 操作系统 | macOS 26.3.1 (a) |
| OpenClaw版本 | OpenClaw 2026.4.26 |
| 系统级Node版本 | v24.15.0 (Homebrew node@24) |
| NVM的Node版本 | v24.11.1 |

## 新手上手建议

1. 先跑通 openclaw gateway status，确认服务正常。

2. 访问 Dashboard http://127.0.0.1:18789 看看界面，熟悉功能

3. 先装安全技能 Skill-Vetter，再按需安装其他 Skills

4. 配置 USER.md，告诉 Agent 你的技术背景（比如 Python/FastAPI 后端开发），让 AI 更懂你

5. 先单 Agent 用熟，再考虑多 Agent 协作

在 `~/.openclaw/openclaw.json` 中可以配置多个 Agent，并分配不同模型。

角色分工建议（混合模型省 40-60% 成本）：

- 路由/编排：Claude Haiku（便宜）
- 代码：Claude Sonnet / Codex
- 长文写作：DeepSeek-R1
- 研究综合：GPT-4.1

## 打开可视化 Dashboard（浏览器访问 http://127.0.0.1:18789）

```bash
openclaw dashboard
```

就是可视化网页界面，OpenClaw 的 Gateway 服务启动后，会在本地开一个 Web 服务，用浏览器访问就能看到图形界面。

侧边栏导航分为四大区域：

### 聊天

| 菜单 | 功能 |
| --- | --- |
| 聊天 | 在网页里直接跟 AI 对话 |

### 控制（运维监控区）

| 菜单 | 功能 |
| --- | --- |
| 概览 | 全局仪表盘，查看系统运行状态、CPU/内存占用等 |
| 频道 | 管理对接的通讯平台（微信、QQ、Telegram、飞书等） |
| 实例 | 当前运行中的 OpenClaw 实例信息（进程状态、端口等） |
| 会话 | 所有历史对话记录，可搜索、回溯、导出 |
| 使用情况 | 从 API 服务商拉取 Token 用量和配额数据 |
| 定时任务 | 设置定时执行的自动化任务 |

### 代理（AI 智能体区）

| 菜单 | 功能 |
| --- | --- |
| 代理 | 管理 AI Agent（创建、编辑、删除），可分配不同角色和模型 |
| 技能 | 查看和管理已安装的 Skills 插件 |
| 节点 | 多机部署时管理各个节点 |
| 梦境 | 后台记忆整合系统，通过浅睡→快速眼动→深睡三阶段将短期记忆筛选后写入长期记忆（实验性功能，默认关闭） |

### 设置

| 菜单 | 功能 |
| --- | --- |
| 配置 | 在线编辑 `openclaw.json`（模型、API Key 等），改完自动生效 |
| 通信 | 配置网关通信参数（端口、绑定地址、安全设置等） |
| 文档 | 链接到官方文档 |

## 设定IDENTITY.md 

1. 为你起名 "ClawClaw"
2. 叫我 "陈培龙" 就好
3. 高效直接风格

## 心跳监测机制:

![心跳监测机制](./images/心跳监测机制.png)

- Tool（18:03） — 系统自动发出了 HEARTBEAT_OK
- ClawClaw（18:03） — Agent 收到心跳后做了响应

这是 OpenClaw 的自动心跳监测机制，作用是：

检测 Agent 是否存活 — Gateway 定期向 Agent 发送心跳信号，Agent 回复表示"我还在"

## 更新openclaw

```bash
openclaw update
```

## openclaw gateway status--查看openclaw网关状态

```bash
(base) peilongchencc@chenpeilongdeMacBook-Pro MyOpenClaw % openclaw gateway status 

🦞 OpenClaw 2026.4.25 (aa36ee6) — Open source means you can see exactly how I judge your config.

│
◇  
Service: LaunchAgent (loaded)
File logs: /tmp/openclaw/openclaw-2026-04-28.log
Command: /opt/homebrew/bin/node /opt/homebrew/lib/node_modules/openclaw/dist/index.js gateway --port 18789
Service file: ~/Library/LaunchAgents/ai.openclaw.gateway.plist
Service env: OPENCLAW_GATEWAY_PORT=18789

Config (cli): ~/.openclaw/openclaw.json
Config (service): ~/.openclaw/openclaw.json

Gateway: bind=loopback (127.0.0.1), port=18789 (service args)
Probe target: ws://127.0.0.1:18789
Dashboard: http://127.0.0.1:18789/
Probe note: Loopback-only gateway; only local clients can connect.

Runtime: running (pid 32312, state active)
Connectivity probe: ok
Capability: read-only

Listening: 127.0.0.1:18789
Troubles: run openclaw status
Troubleshooting: https://docs.openclaw.ai/troubleshooting
(base) peilongchencc@chenpeilongdeMacBook-Pro MyOpenClaw % 
```

## openclaw doctor --repair

自动修复服务配置

```bash
(base) peilongchencc@chenpeilongdeMacBook-Pro MyOpenClaw % openclaw doctor --repair

🦞 OpenClaw 2026.4.25 (aa36ee6) — Say "stop" and I'll stop—say "ship" and we'll both learn a lesson.

▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
██░▄▄▄░██░▄▄░██░▄▄▄██░▀██░██░▄▄▀██░████░▄▄▀██░███░██
██░███░██░▀▀░██░▄▄▄██░█░█░██░█████░████░▀▀░██░█░█░██
██░▀▀▀░██░█████░▀▀▀██░██▄░██░▀▀▄██░▀▀░█░██░██▄▀▄▀▄██
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
                  🦞 OPENCLAW 🦞                    
 
┌  OpenClaw doctor
│
◇  Plugin registry ────────────────────────────────────────────╮
│                                                              │
│  Plugin registry refreshed: 66/112 enabled plugins indexed.  │
│                                                              │
├──────────────────────────────────────────────────────────────╯
│
◇  State integrity ─────────────────────────────────────────────────────────────────────────╮
│                                                                                           │
│  - OAuth dir not present (~/.openclaw/credentials). Skipping create because no            │
│    WhatsApp/pairing channel config is active.                                             │
│  - Found 1 orphan transcript file in ~/.openclaw/agents/main/sessions.                    │
│    These .jsonl files are no longer referenced by sessions.json, so they are not part of  │
│    any active session history.                                                            │
│    Doctor can archive them safely by renaming each file to *.deleted.<timestamp>.         │
│    Examples: 1d4d8b25-ef21-4f43-8315-977ff9c309b4.trajectory.jsonl                        │
│                                                                                           │
├───────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  Doctor changes ──────────────────────────────────────────────────────────────────────╮
│                                                                                       │
│  - Archived 1 orphan transcript file in ~/.openclaw/agents/main/sessions as .deleted  │
│    timestamped backups.                                                               │
│                                                                                       │
├───────────────────────────────────────────────────────────────────────────────────────╯
│
◇  Gateway service config ────────────────────────────────────────────────────────────╮
│                                                                                     │
│  - Gateway service PATH includes version managers or package managers; recommend a  │
│    minimal PATH. (/Users/peilongchencc/.nvm/versions/node/v24.11.1/bin)             │
│  - Gateway service uses Node from a version manager; it can break after upgrades.   │
│    (/Users/peilongchencc/.nvm/versions/node/v24.11.1/bin/node)                      │
│                                                                                     │
├─────────────────────────────────────────────────────────────────────────────────────╯

Installed LaunchAgent: /Users/peilongchencc/Library/LaunchAgents/ai.openclaw.gateway.plist
Logs: /Users/peilongchencc/.openclaw/logs/gateway.log
│
◇  Security ─────────────────────────────────╮
│                                            │
│  - No channel security warnings detected.  │
│  - Run: openclaw security audit --deep     │
│                                            │
├────────────────────────────────────────────╯
│
◇  Skills status ────────────╮
│                            │
│  Eligible: 8               │
│  Missing requirements: 48  │
│  Blocked by allowlist: 0   │
│                            │
├────────────────────────────╯
│
◇  Plugins ──────╮
│                │
│  Loaded: 66    │
│  Imported: 0   │
│  Disabled: 46  │
│  Errors: 0     │
│                │
├────────────────╯
│
◇  
Agents: main (default)
Heartbeat interval: 30m (main)
Session store (main): /Users/peilongchencc/.openclaw/agents/main/sessions/sessions.json (1 entries)
- agent:main:main (3m ago)
│
└  Doctor complete.

(base) peilongchencc@chenpeilongdeMacBook-Pro MyOpenClaw %
```

## OpenClaw Gateway 使用 NVM Node 的修复指南

### 错误信息示例:

运行 `openclaw gateway status`，如果看到类似下列警告就需要修复：

```
Service config issue: Gateway service PATH not standard;
/Users/peilongchencc/.nvm/versions/node/v24.11.1/bin/node   ← 不稳定，含具体版本
```

### 背景：为什么会出现这个问题？

macOS 的 LaunchAgent（系统级后台服务）和个人的终端 shell 是**两个完全独立的运行环境**：

- **终端 shell**：启动时加载 `~/.zshrc`，NVM 把自己管理的 Node 注入 PATH 最前面
- **LaunchAgent**：系统启动时运行，**不加载任何 shell 配置文件**，只依赖 plist 文件里写死的路径

NVM 是为 "人机交互的终端" 设计的，Homebrew/系统 Node 是为 "无人值守的后台服务" 设计的。

两者的使用场景不同，OpenClaw 的 gateway 服务属于后者，所以推荐系统级 Node。

现在由于加载了错误的Node路径，导致OpenClaw的gateway服务报警，需要修复。

### 修复方案:

临时将 Homebrew Node 置于 PATH 最前面，再重装 gateway 服务 / 执行修复命令。

注意: 不需要永久将 Homebrew 的 node 置于 PATH 首位，只需要临时修复即可。这是不建议的操作。日常开发用 NVM 管理 Node，可以随时切换版本（nvm use 20、nvm use 24 等），灵活性更高。

```bash
# 卸载当前 gateway 服务
openclaw gateway uninstall
# 加载系统级Node，并安装新的gateway服务，注意：&& 保证前一步成功后才执行安装
export PATH="/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:$PATH" && openclaw gateway install
```

```bash
# 临时修复命令
export PATH="/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:$PATH" && openclaw doctor --repair
```

一行搞定，&& 保证前一步成功后才执行修复。

### 验证修复效果

```bash
openclaw gateway status
```

此时不报错，说明修复成功。