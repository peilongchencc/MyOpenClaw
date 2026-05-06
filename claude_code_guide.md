# Claude Code 使用指南（接入千问模型）

本文整理自首次安装配置 Claude Code 的完整过程，涵盖安装、配置千问模型、使用方法和省 Token 技巧。

- [Claude Code 使用指南（接入千问模型）](#claude-code-使用指南接入千问模型)
  - [一、Claude Code 是什么？](#一claude-code-是什么)
  - [二、安装](#二安装)
    - [2.1 环境要求](#21-环境要求)
    - [2.2 安装命令](#22-安装命令)
    - [2.3 验证安装](#23-验证安装)
  - [三、配置千问模型（无需 Claude API Key）](#三配置千问模型无需-claude-api-key)
    - [3.1 原理说明](#31-原理说明)
    - [3.2 获取阿里云百炼 API Key](#32-获取阿里云百炼-api-key)
    - [3.3 创建 settings.json](#33-创建-settingsjson)
    - [3.4 settings.json 各字段详解](#34-settingsjson-各字段详解)
    - [3.5 跳过 Anthropic 登录验证](#35-跳过-anthropic-登录验证)
    - [3.6 验证配置](#36-验证配置)
  - [四、可用的千问模型一览](#四可用的千问模型一览)
    - [4.1 模型选型建议](#41-模型选型建议)
    - [4.2 按任务类型推荐](#42-按任务类型推荐)
  - [五、基本使用](#五基本使用)
    - [5.1 启动与退出](#51-启动与退出)
    - [5.2 `--` 参数 vs `/` 命令：什么时候用哪个？](#52---参数-vs--命令什么时候用哪个)
    - [5.3 常用斜杠命令（`/` 命令）](#53-常用斜杠命令-命令)
    - [5.4 常用启动参数（`--` 参数）](#54-常用启动参数---参数)
    - [5.5 对话示例](#55-对话示例)
    - [5.6 `/init` 与 CLAUDE.md](#56-init-与-claudemd)
  - [六、省 Token 技巧](#六省-token-技巧)
  - [七、配置修改指南](#七配置修改指南)
    - [7.1 切换模型](#71-切换模型)
    - [7.2 切换地域](#72-切换地域)
    - [7.3 使用 Shell 函数快速切换](#73-使用-shell-函数快速切换)
  - [八、注意事项](#八注意事项)
  - [九、与 Cursor 的关系](#九与-cursor-的关系)

---

## 一、Claude Code 是什么？

Claude Code 是 Anthropic 推出的**命令行 AI 编程助手**（CLI 工具），直接运行在终端中。它不是代码补全工具，而是一个**代理式（Agentic）AI 工程工具**，核心能力包括：

- **理解整个项目**：自动扫描项目目录，理解代码结构和依赖关系
- **直接操作文件**：跨文件创建、编辑、重命名、删除
- **执行 Shell 命令**：运行测试、安装依赖、执行构建等
- **Git 操作**：分析 diff、写 commit message、处理 merge conflict
- **多轮对话**：持续记住上下文，进行深度调试和迭代开发

**类比**：如果 Cursor 是"坐在 IDE 里的 AI 助手"，Claude Code 就是"坐在终端里的 AI 工程师"。

---

## 二、安装

### 2.1 环境要求

| 要求 | 说明 |
| --- | --- |
| 操作系统 | macOS 10.15+、Ubuntu 20.04+、Windows 10/11 |
| 内存 | 至少 4GB RAM（推荐 8GB+） |
| 网络 | 需要互联网连接 |

### 2.2 安装命令

```bash
# macOS / Linux（推荐方式）
curl -fsSL https://claude.ai/install.sh | bash

# macOS Homebrew（不会自动更新）
brew install --cask claude-code

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex
```

安装后，可执行文件位于 `~/.local/bin/claude`。

### 2.3 验证安装

```bash
claude --version
# 输出示例: 2.1.129
```

---

## 三、配置千问模型（无需 Claude API Key）

### 3.1 原理说明

Claude Code 本身只是一个"执行框架"，它的能力取决于接入的大模型。阿里云百炼（DashScope）提供了 **Anthropic API 兼容接口**，因此 Claude Code 可以直接对接千问系列模型，**不需要 Anthropic/Claude 的 API Key**。

### 3.2 获取阿里云百炼 API Key

1. 打开 [阿里云百炼控制台](https://bailian.console.aliyun.com/)
2. 注册/登录阿里云账号
3. 在百炼平台中创建 API Key（参考[官方文档](https://help.aliyun.com/zh/model-studio/get-api-key)）

### 3.3 创建 settings.json

配置文件路径：`~/.claude/settings.json`

```bash
# 如果 .claude 目录不存在，先创建
mkdir -p ~/.claude
```

编辑 `~/.claude/settings.json`，写入以下内容：

```json
{
    "env": {
        "ANTHROPIC_AUTH_TOKEN": "sk-你的百炼API-Key",
        "ANTHROPIC_BASE_URL": "https://dashscope.aliyuncs.com/apps/anthropic",
        "ANTHROPIC_MODEL": "qwen3.6-plus",
        "ANTHROPIC_SMALL_FAST_MODEL": "qwen3.6-flash",
        "ANTHROPIC_DEFAULT_HAIKU_MODEL": "qwen3.6-flash",
        "ANTHROPIC_DEFAULT_SONNET_MODEL": "qwen3.6-plus",
        "ANTHROPIC_DEFAULT_OPUS_MODEL": "qwen3.6-plus",
        "CLAUDE_CODE_SUBAGENT_MODEL": "qwen3.6-plus"
    }
}
```

### 3.4 settings.json 各字段详解

`settings.json` 的 `env` 对象中定义的是**环境变量**，Claude Code 启动时会自动加载。以下是每个字段的含义和修改指南：

#### 认证与连接

| 字段 | 含义 | 修改场景 |
| --- | --- | --- |
| `ANTHROPIC_AUTH_TOKEN` | 你的 API Key，用于调用模型服务 | 更换 API Key 时修改 |
| `ANTHROPIC_BASE_URL` | 模型服务的 API 端点地址 | 切换服务商或地域时修改 |

> **注意**：`ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_API_KEY` 只能选一个，不要同时设置，否则可能冲突。推荐使用 `ANTHROPIC_AUTH_TOKEN`。

#### 模型配置

Claude Code 内部会根据任务类型自动选择不同级别的模型：

| 字段 | 对应角色 | 用途 | 修改建议 |
| --- | --- | --- | --- |
| `ANTHROPIC_MODEL` | 主模型 | `claude` 命令启动后默认使用的模型 | 想换默认模型时改这里 |
| `ANTHROPIC_SMALL_FAST_MODEL` | 快速模型 | 用于轻量级、低延迟的任务 | 追求速度和低成本时用 flash 系列 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku 级别 | 简单任务：语法检查、文件搜索、格式化 | 用最便宜的模型即可 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet 级别 | 日常任务：代码编写、功能实现、Bug 修复 | 建议用 plus 系列，性价比最高 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus 级别 | 复杂任务：架构设计、复杂推理、大型重构 | 追求最强能力时用 max 系列 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 子 Agent 模型 | Claude Code 内部派生子任务时使用的模型 | 通常和 Sonnet 保持一致 |

> 这些名称（Haiku/Sonnet/Opus）源自 Claude 官方的模型等级命名，在接入千问时实际都指向千问的模型，只是按任务复杂度做了分级。

#### 配置示例：按需分级

如果你想精细控制成本，可以给不同复杂度的任务分配不同模型：

```json
{
    "env": {
        "ANTHROPIC_AUTH_TOKEN": "sk-你的API-Key",
        "ANTHROPIC_BASE_URL": "https://dashscope.aliyuncs.com/apps/anthropic",
        "ANTHROPIC_MODEL": "qwen3.6-plus",
        "ANTHROPIC_SMALL_FAST_MODEL": "qwen3.6-flash",
        "ANTHROPIC_DEFAULT_HAIKU_MODEL": "qwen3.6-flash",
        "ANTHROPIC_DEFAULT_SONNET_MODEL": "qwen3.6-plus",
        "ANTHROPIC_DEFAULT_OPUS_MODEL": "qwen3.6-max-preview",
        "CLAUDE_CODE_SUBAGENT_MODEL": "qwen3.6-plus"
    }
}
```

这样：简单任务用 flash（便宜快速）→ 日常任务用 plus（均衡）→ 复杂任务用 max（最强）。

### 3.5 跳过 Anthropic 登录验证

编辑 `~/.claude.json`，添加 `hasCompletedOnboarding` 字段：

```json
{
  "hasCompletedOnboarding": true,
  "installMethod": "native",
  ...其他已有字段保持不变...
}
```

> `hasCompletedOnboarding` 必须是顶层字段，不能嵌套在其他字段里。没有这个设置，Claude Code 启动时会尝试连接 Anthropic 官方服务，导致报错 `Unable to connect to Anthropic services`。

### 3.6 验证配置

**必须打开一个新的终端窗口**（旧终端不会加载新配置），然后：

```bash
cd ~/Desktop/MyOpenClaw  # 进入你的项目目录
claude                    # 启动 Claude Code
```

首次进入会弹出安全确认，选择 "Yes, I trust this folder" 后按回车。

进入对话界面后，输入 `/status` 确认配置是否正确：

```
Auth token:          ANTHROPIC_AUTH_TOKEN        ← 应该显示这个
Anthropic base URL:  https://dashscope.aliyuncs.com/apps/anthropic  ← 百炼地址
Model:               qwen3.6-plus               ← 千问模型
```

---

## 四、可用的千问模型一览

### 4.1 模型选型建议

| 模型系列 | 模型名称 | 特点 |
| --- | --- | --- |
| **千问 Max** | `qwen3.6-max-preview`、`qwen3-max` | 最强推理能力，支持思考模式 |
| **千问 Plus** | `qwen3.6-plus`（推荐日常使用）、`qwen3.5-plus` | 能力与成本均衡 |
| **千问 Flash** | `qwen3.6-flash`（推荐简单任务）、`qwen3.5-flash` | 速度快、成本低 |
| **千问 Coder** | `qwen3-coder-next`、`qwen3-coder-plus` | 专门优化的编码模型 |
| **千问 VL** | `qwen3-vl-plus`、`qwen3-vl-flash` | 支持图片理解（视觉模型） |
| **千问开源** | `qwen3.5-397b-a17b`、`qwen3.5-27b` | 开源模型 |
| **第三方模型** | `deepseek-v4-pro`、`kimi-k2.5`、`glm-5.1` 等 | 百炼也代理了其他厂商的模型 |

### 4.2 按任务类型推荐

| 任务类型 | 推荐模型 |
| --- | --- |
| 复杂推理（架构设计、算法实现） | `qwen3.6-plus`、`qwen3.6-max-preview`、`qwen3-coder-next` |
| 日常编码（写函数、改 Bug） | `qwen3.6-plus`、`qwen3-coder-plus` |
| 简单任务（注释、搜索、格式化） | `qwen3.6-flash`、`qwen3-coder-next` |

---

## 五、基本使用

### 5.1 启动与退出

```bash
# 启动（在项目目录下运行）
cd ~/Desktop/MyOpenClaw
claude

# 退出（在对话界面中输入以下任一）
exit
/exit
# 或按 Ctrl+C
```

### 5.2 `--` 参数 vs `/` 命令：什么时候用哪个？

Claude Code 有两套指令体系，初次使用容易混淆，核心区别是：**没进去用 `--`，进去了用 `/`**。

#### `--` 参数：在终端中**启动之前**用

`--` 参数是启动 `claude` 命令时的参数，用于控制本次会话的初始设置。就像 `python --version`、`git --help` 一样，是给程序传的启动参数。

```bash
# 在终端中输入（还没进入 Claude Code 对话界面）：
claude --model qwen3.6-flash        # 指定模型启动
claude -c                            # 继续上次对话
claude -p "写一个hello world"         # 非交互模式，直接输出
```

#### `/` 命令：在对话界面中**启动之后**用

`/` 命令是在已经进入 Claude Code 交互界面后使用的，用于临时调整当前会话的状态。它不是给 AI 的问题，而是对工具本身的控制指令。

```
# 已经进入了 Claude Code（看到 ❯ 提示符），在里面输入：
/status                # 查看配置
/model qwen3.6-flash   # 临时切换模型
/compact               # 压缩对话
/clear                 # 清除上下文
```

#### 对比总结

| 维度 | `--` 参数 | `/` 命令 |
| --- | --- | --- |
| **在哪里用** | 终端命令行（启动前） | Claude Code 对话界面（启动后） |
| **什么时候用** | 启动 `claude` 的那一刻 | 已经在和 AI 对话的过程中 |
| **影响范围** | 决定整个会话的初始设置 | 临时调整当前会话 |
| **语法** | `claude --参数名 值` | `/命令名 值` |

#### 完整操作流程示例

```bash
# 第一步：在终端中，用 -- 参数启动
claude --model qwen3.6-plus

# 第二步：进入对话界面后，看到 ❯ 提示符
❯ /status                          # 用 / 命令查看状态
❯ 帮我写一个FastAPI接口              # 正常和 AI 对话
❯ /model qwen3.6-flash             # 用 / 命令临时切换为更便宜的模型
❯ 帮我加个注释                      # 继续对话（此时用的是 flash 模型）
❯ /compact                         # 用 / 命令压缩对话历史
❯ exit                             # 退出
```

### 5.3 常用斜杠命令（`/` 命令）

在对话界面中输入以下命令：

| 命令 | 作用 |
| --- | --- |
| `/status` | 查看当前配置（模型、API Key、Base URL） |
| `/model 模型名` | 临时切换模型，如 `/model qwen3.6-flash` |
| `/compact` | 压缩对话历史，节省 Token |
| `/clear` | 清除所有上下文，开始全新对话 |
| `/init` | 创建 CLAUDE.md 项目说明文件（告诉 AI 项目背景） |
| `/help` | 查看所有可用命令 |

### 5.4 常用启动参数（`--` 参数）

```bash
# 指定模型启动
claude --model qwen3.6-flash

# 继续上次对话
claude -c
# 或 claude --continue

# 恢复历史会话（交互式选择）
claude -r
# 或 claude --resume

# 非交互模式（直接输出结果，适合脚本/管道）
claude -p "帮我写一个 Python hello world"

# 给会话命名
claude -n "重构API模块"
```

### 5.5 对话示例

```
❯ 帮我看看这个项目的目录结构
❯ 解释一下 README.md 的内容
❯ 帮我用 FastAPI 写一个健康检查接口
❯ 分析 git log，帮我写一份本周的变更总结
❯ 帮我把 requirements.txt 中的 requests 替换为 aiohttp
```

### 5.6 `/init` 与 CLAUDE.md

在对话界面中输入 `/init`，Claude Code 会**自动扫描当前项目的目录结构、代码文件、配置文件等**，然后在项目根目录下生成一个 `CLAUDE.md` 文件，并自动写入内容。你不需要手动编写，生成后可以自己修改补充。

#### CLAUDE.md 是什么？

`CLAUDE.md` 相当于给 AI 写的**项目说明书**。Claude Code 每次启动时会自动读取项目目录下的 `CLAUDE.md`，从而一上来就理解你的项目背景，不需要每次都重新扫描和猜测。

#### 通常包含哪些内容？

- 项目背景和用途
- 技术栈（如 Python + FastAPI）
- 代码规范（如注释风格、日志库偏好）
- 目录结构说明
- 常用命令（如何运行、如何测试、如何部署）

#### 作用范围

`CLAUDE.md` 是**针对当前项目目录**的，每个项目有自己独立的 `CLAUDE.md`，互不影响：

```bash
cd ~/Desktop/MyOpenClaw
claude
❯ /init    # → 生成 ~/Desktop/MyOpenClaw/CLAUDE.md

cd ~/Desktop/MyFastAPI
claude
❯ /init    # → 生成 ~/Desktop/MyFastAPI/CLAUDE.md
```

#### 是否必须执行？

**不是必须的**。不运行 `/init`，Claude Code 照样能用，只是它对项目的理解完全依赖于实时扫描文件。写了 `CLAUDE.md` 后，AI 的回答会更精准，也能减少不必要的文件扫描（间接节省 Token）。

建议在用熟 Claude Code 之后，再给常用项目执行 `/init` 生成 `CLAUDE.md`。

#### 文件位置

`CLAUDE.md` 直接放在**项目根目录**下，和 `README.md` 同级，不是隐藏文件，也不在任何 `.` 开头的文件夹里：

```
MyOpenClaw/
├── CLAUDE.md          ← 就在这里，和 README.md 同级
├── README.md
├── skills_learning_notes.md
└── ...
```

这样设计是为了方便提交到 Git 仓库，团队成员 clone 项目后 Claude Code 就能直接读取。如果不想提交，可以把它加到 `.gitignore` 里。

#### 如何更新 CLAUDE.md？

随着项目不断迭代，`CLAUDE.md` 的内容可能会过时。有三种方式更新：

**方式一：重新执行 `/init`**

在 Claude Code 对话界面中再次输入 `/init`，它会重新扫描项目并询问你是否覆盖现有内容。

**方式二：手动编辑**

`CLAUDE.md` 就是普通的 Markdown 文件，直接在 Cursor 或任何编辑器中修改即可。这种方式更常用，因为你可以精确地加入对 AI 最有帮助的信息。

**方式三：让 Claude Code 帮你更新**

在对话中直接告诉它要补充什么，它会自动编辑 `CLAUDE.md`：

```
❯ 帮我更新 CLAUDE.md，加上我们使用 FastAPI 构建接口、日志用 loguru、HTTP 请求用 aiohttp
```

> 实际使用中，方式二和方式三更推荐。`/init` 自动生成的内容偏通用，而你在使用过程中逐渐积累的项目特定信息（如代码规范、技术栈偏好）手动加进去效果更好。

---

## 六、省 Token 技巧

Claude Code 会扫描整个项目目录来理解代码，Token 消耗远高于普通对话。以下方法可以有效控制成本：

1. **在具体项目目录中启动**：不要在 `~` 目录启动，否则会扫描大量无关文件
2. **精确描述需求**：避免模糊指令（如"帮我优化代码"），说清具体文件和目标
3. **用 `/compact` 压缩对话**：对话过长时手动压缩，减少上下文累积
4. **新任务前用 `/clear` 重置**：开始全新任务时清除旧上下文
5. **简单任务用 Flash 模型**：`/model qwen3.6-flash`，完成后再切回来
6. **分解复杂任务**：大任务拆成小步骤，每步明确具体

---

## 七、配置修改指南

所有修改都在 `~/.claude/settings.json` 中进行，修改后**必须打开新终端窗口**才能生效。

### 7.1 切换模型

只需修改 `ANTHROPIC_MODEL` 字段：

```json
"ANTHROPIC_MODEL": "qwen3-coder-next"
```

如果想全部切换，把所有模型字段都改掉。

### 7.2 切换地域

百炼提供两个地域：

| 地域 | Base URL |
| --- | --- |
| 华北2（北京） | `https://dashscope.aliyuncs.com/apps/anthropic` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/apps/anthropic` |

> 注意：API Key 必须和地域匹配，北京地域的 Key 不能用于新加坡端点。

### 7.3 使用 Shell 函数快速切换

如果你经常需要在不同模型间切换，可以在 `~/.zshrc` 中定义快捷函数：

```bash
# 使用千问 Plus（日常使用）
qwen_plus() {
    export ANTHROPIC_BASE_URL="https://dashscope.aliyuncs.com/apps/anthropic"
    export ANTHROPIC_AUTH_TOKEN="sk-你的API-Key"
    export ANTHROPIC_MODEL="qwen3.6-plus"
    echo "已切换到 qwen3.6-plus"
}

# 使用千问 Flash（省钱模式）
qwen_flash() {
    export ANTHROPIC_BASE_URL="https://dashscope.aliyuncs.com/apps/anthropic"
    export ANTHROPIC_AUTH_TOKEN="sk-你的API-Key"
    export ANTHROPIC_MODEL="qwen3.6-flash"
    echo "已切换到 qwen3.6-flash"
}
```

使用方式：先在终端中输入 `qwen_flash`，再运行 `claude`。

---

## 八、注意事项

1. **能力差异**：国内模型在日常编码任务上表现不错，但在大型项目全局理解、复杂系统调试方面与 Claude 官方模型仍有差距。适合处理边界清晰的明确任务。

2. **API Key 安全**：`~/.claude/settings.json` 中包含你的 API Key，不要将这个文件提交到 Git 仓库。

3. **鉴权变量冲突**：`ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 只能选一个，不要同时设置。

4. **配置修改后必须新开终端**：修改 `settings.json` 后，旧终端窗口不会自动加载新配置。

5. **首次进入项目目录的安全确认**：Claude Code 首次进入某个目录时会问你是否信任该目录，选 "Yes" 即可，后续不会再问。

---

## 九、与 Cursor 的关系

Claude Code 和 Cursor 是**互补关系**，不冲突：

| 维度 | Cursor | Claude Code |
| --- | --- | --- |
| 工作环境 | IDE 内（图形界面） | 终端（命令行） |
| 工作方式 | 在编辑器中辅助编码 | 自主执行多步骤任务 |
| 文件操作 | 在 IDE 中编辑 | 直接操作文件系统 |
| Shell 命令 | 不直接执行 | 可以直接运行 |
| 适合场景 | 写代码、改代码 | 项目级任务、自动化、运维 |

两者可以同时使用：在 Cursor 中写代码，在终端中用 Claude Code 处理项目级任务。Claude Code 启动时如果检测到 Cursor/VS Code，还会自动连接 IDE 扩展，方便你在 IDE 中查看 Claude Code 对文件的修改。
