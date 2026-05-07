# Skills 实战指南

> 整理时间：2026-05-07
>
> 覆盖工具：Claude Code、Cursor、OpenClaw

- [Skills 实战指南](#skills-实战指南)
  - [一、什么是 Skills](#一什么是-skills)
  - [二、为什么在项目目录会"自动触发"](#二为什么在项目目录会自动触发)
  - [三、Skill 放哪里（各工具路径）](#三skill-放哪里各工具路径)
    - [Claude Code](#claude-code)
    - [Cursor](#cursor)
    - [OpenClaw](#openclaw)
    - [跨工具兼容性](#跨工具兼容性)
    - [workspace 对比](#workspace-对比)
  - [四、SKILL.md 的结构](#四skillmd-的结构)
    - [1. YAML Frontmatter（元数据）](#1-yaml-frontmatter元数据)
    - [2. Markdown Body（正文指引）](#2-markdown-body正文指引)
    - [最小模板](#最小模板)
    - [完整示例](#完整示例)
  - [五、自己写 Skill 的步骤](#五自己写-skill-的步骤)
  - [六、让 Skill 生效](#六让-skill-生效)
  - [七、验证 Skill 是否加载](#七验证-skill-是否加载)
    - [通用方法：暗号测试法](#通用方法暗号测试法)
    - [OpenClaw 专属](#openclaw-专属)
  - [八、安装第三方 Skills 包](#八安装第三方-skills-包)
  - [九、Skills vs Rules](#九skills-vs-rules)
  - [十、常见问题 FAQ](#十常见问题-faq)
    - [Q: Cursor 为什么能识别 `.claude/skills/` 下的内容？](#q-cursor-为什么能识别-claudeskills-下的内容)
    - [Q: Claude Code 支持 `.agents/skills/` 路径吗？](#q-claude-code-支持-agentsskills-路径吗)
    - [Q: OpenClaw 的 skill 在终端和 Dashboard 中是否一样？](#q-openclaw-的-skill-在终端和-dashboard-中是否一样)
    - [Q: 第三方 skills 包能不能打散放置？](#q-第三方-skills-包能不能打散放置)
    - [Q: OpenClaw 自定义 skill 不被识别怎么办？](#q-openclaw-自定义-skill-不被识别怎么办)
  - [十一、官方文档链接](#十一官方文档链接)

---

## 一、什么是 Skills

Skills 本质上就是一组 `SKILL.md` 文件，是写给 AI 助手（Claude Code / Cursor / OpenClaw）看的「操作手册」。

> **Skills = 给 AI 的操作手册**
>
> 告诉 AI："当用户说了 XXX 这类话的时候，你应该按照这个文档的指引去操作。"

每个 `SKILL.md` 文件包含：

1. **触发条件** — 用户说什么话时应该激活这个 skill
2. **具体操作步骤** — AI 应该调用哪些 API、怎么组织输出
3. **示例** — 输入/输出的范例

> 想深入理解 Skills 的概念、原理和演化脉络，可阅读 [skills_learning_notes.md](skills_learning_notes.md)。

---

## 二、为什么在项目目录会"自动触发"

Claude Code / Cursor 在启动时，会**自动递归扫描**当前工作目录下的特定路径，寻找 `SKILL.md` 文件。

整个流程如下：

```
你在某个项目目录打开 Cursor / Claude Code
         |
         v
AI 自动扫描 .claude/skills/ 和 .cursor/skills/ 等目录
         |
         v
发现 SKILL.md 文件
         |
         v
将这些 skills 注册到「可用技能列表」
         |
         v
你输入一条消息
         |
         v
AI 判断：你的消息是否匹配某个 skill 的触发条件？
         |
    ┌────┴────┐
    是        否
    |         |
    v         v
读取对应     正常回答
SKILL.md
并按指引执行
```

---

## 三、Skill 放哪里（各工具路径）

### Claude Code

| 作用域 | 路径 | 说明 |
|--------|------|------|
| 用户级 | `~/.claude/skills/<技能名>/SKILL.md` | 个人全局，所有项目可用 |
| 项目级 | `<项目>/.claude/skills/<技能名>/SKILL.md` | 仅当前项目，可 git 提交共享给团队 |
| 插件级 | `<插件>/skills/<技能名>/SKILL.md` | 插件启用时可用 |

优先级：企业 > 用户 > 项目。同名 skill 高优先级覆盖低优先级。

> Claude Code **只支持** `.claude/skills/` 和 `.claude/commands/`，**不支持** `.agents/skills/` 等路径。

### Cursor

| 作用域 | 路径 | 说明 |
|--------|------|------|
| 项目级 | `<项目>/.cursor/skills/` | Cursor 专属路径 |
| 项目级 | `<项目>/.agents/skills/` | 通用路径 |
| 用户级 | `~/.cursor/skills/` | 个人全局 |
| 用户级 | `~/.agents/skills/` | 通用路径 |
| 兼容 | `<项目>/.claude/skills/` | Cursor 也能识别 Claude Code 的路径 |
| 兼容 | `<项目>/.codex/skills/` | Cursor 也能识别 Codex 的路径 |

### OpenClaw

| 优先级 | 路径 | 说明 |
|--------|------|------|
| 1（最高） | `~/.openclaw/workspace/skills/` | workspace 级别（Dashboard 默认 workspace） |
| 2 | `~/.openclaw/workspace/.agents/skills/` | workspace 内的 agent skills |
| 3 | `~/.agents/skills/` | 个人全局 agent skills |
| 4 | `~/.openclaw/skills/` | 个人全局 OpenClaw skills |
| 5 | 内置 bundled | 随安装自带 |
| 6（最低） | `openclaw.json` → `skills.load.extraDirs` | 额外自定义目录 |

> OpenClaw 的 workspace 默认是 `~/.openclaw/workspace`（不跟随终端 cd 目录）。

### 跨工具兼容性

**不存在一个路径能让三个工具同时识别。** 各工具兼容情况如下：

| 路径 | Claude Code | Cursor | OpenClaw |
|------|:-----------:|:------:|:--------:|
| `.claude/skills/` | ✅ 原生 | ✅ 兼容 | ❌ |
| `.cursor/skills/` | ❌ | ✅ 原生 | ❌ |
| `.agents/skills/` | ❌ | ✅ 支持 | ✅ 支持（workspace 下） |
| `.codex/skills/` | ❌ | ✅ 兼容 | ❌ |

- 想让 **Claude Code + Cursor** 都能用：放 `<项目>/.claude/skills/`（Cursor 兼容此路径）
- 想让 **Cursor + OpenClaw** 都能用：分别在两个位置各放一份

### workspace 对比

| 工具 | workspace 由什么决定 |
|------|---------------------|
| Cursor | 你用 Cursor 打开了哪个项目文件夹 |
| Claude Code | 你在终端 `cd` 到了哪个目录 |
| OpenClaw | `openclaw.json` 中配置的固定路径（默认 `~/.openclaw/workspace`） |

---

## 四、SKILL.md 的结构

每个 `SKILL.md` 文件由两部分组成：

### 1. YAML Frontmatter（元数据）

```yaml
---
name: my-skill-name
description: >
  一句话描述这个 skill 干什么。
  Triggers on: "触发词1", "触发词2", "触发词3"
---
```

必填字段：

| 字段 | 说明 |
|------|------|
| `name` | 技能名称，小写字母+数字+连字符，最多 64 字符 |
| `description` | 技能描述 + 触发词，告诉 AI 什么时候使用这个 skill |

可选字段（高级用法）：`globs`、`user-invocable`、`disable-model-invocation`、`agent`、`allowed-tools`、`context`、`model`、`effort` 等。

### 2. Markdown Body（正文指引）

```markdown
# 标题

## When to use
- 什么场景下用

## Prerequisites
- 前置条件（API Key 等）

## Instructions
- 具体让 AI 做什么

## Example
- 输入/输出示例

## Output Formatting
- 输出格式建议
```

### 最小模板

```markdown
---
name: my-skill-name
description: >
  一句话描述这个 skill 干什么。
  Triggers on: "触发词1", "触发词2"
---

# Skill 标题

## When to use
- 什么场景下用

## Instructions
- 具体让 AI 做什么
```

### 完整示例

```markdown
---
name: alphagbm-alert
description: |
  Set price, IV, or activity-based alerts with contextual notifications.
  Triggers: "alert me when AAPL IV rank above 80", "notify if NVDA drops below 850",
  "earnings alert for TSLA", "set price alert", "my alerts", "delete alert"
globs:
  - "mock-data/alert/**"
---

# AlphaGBM Alerts

Set intelligent alerts based on price, IV rank, unusual activity...

## What This Skill Does

| Alert Type | Description |
|------------|-------------|
| IV Rank Threshold | Fires when IV rank crosses above or below a specified level |
| Price Level | Fires when price breaks through support or resistance |
...

## API Endpoint

GET    /api/user/alerts
POST   /api/user/alerts
DELETE /api/user/alerts/{alert_id}
```

---

## 五、自己写 Skill 的步骤

**第一步：创建目录**

```bash
# Claude Code（用户级）
mkdir -p ~/.claude/skills/my-skill/

# Claude Code（项目级）
mkdir -p .claude/skills/my-skill/

# Cursor
mkdir -p .cursor/skills/my-skill/

# OpenClaw（Dashboard 模式）
mkdir -p ~/.openclaw/workspace/skills/my-skill/
```

**第二步：创建 SKILL.md**

写入你想让 AI 遵循的指引（参考上方模板）。

**第三步：让工具识别**（见下一节）

---

## 六、让 Skill 生效

| 工具 | 操作 |
|------|------|
| Claude Code | 重新启动 Claude Code |
| Cursor | 重新打开项目 |
| OpenClaw | 聊天中输入 `/new` 或 `openclaw gateway restart` |

---

## 七、验证 Skill 是否加载

### 通用方法：暗号测试法

创建一个包含荒谬暗号的测试 skill（如"西瓜炒芒果"），触发后看输出中是否包含暗号。这样能 100% 确认 skill 是否被加载执行了。

### OpenClaw 专属

```bash
openclaw skills list
# 查看 Source 列是否显示你的 skill
```

Source 列含义：
- `openclaw-workspace` → 来自 `<workspace>/skills/`
- `openclaw-bundled` → 内置 skill
- `openclaw-extra` → 来自插件扩展

---

## 八、安装第三方 Skills 包

```bash
# Claude Code，将第三方技能包克隆到当前项目目录下的 .claude/skills/（如果没有则创建） 目录下
git clone https://github.com/xxx/skills.git .claude/skills/包名

# Cursor，将第三方技能包克隆到当前项目目录下的 .cursor/skills/（如果没有则创建） 目录下
git clone https://github.com/xxx/skills.git .cursor/skills/包名

# OpenClaw（社区 skill）
openclaw skills install <skill-name>
# 或在 Dashboard 中一键安装
```

> 第三方包内部通常有互相引用的路径（mock-data、Related Skills 等），**不要打散它的目录结构**，保持整体作为一个单元使用。后续更新只需在该目录下 `git pull`。

---

## 九、Skills vs Rules

| | Skills | Rules |
|--|--------|-------|
| 用途 | 多步骤工作流、详细操作指引 | 简短的编码规范 |
| 触发方式 | 按需触发（AI 根据描述判断） | 始终生效 |
| 文件 | `SKILL.md` | 各工具有不同的 rules 文件 |
| 长度 | 可以很长，包含 API 说明、示例等 | 通常简短，几条规则 |
| Token 消耗 | 仅在触发时加载 | 每次对话都占用上下文 |

---

## 十、常见问题 FAQ

### Q: Cursor 为什么能识别 `.claude/skills/` 下的内容？

A: Cursor 官方做了兼容支持，会额外扫描 `.claude/skills/` 和 `.codex/skills/`。

### Q: Claude Code 支持 `.agents/skills/` 路径吗？

A: **不支持。** Claude Code 只支持 `.claude/skills/` 和 `.claude/commands/`（官方文档：https://code.claude.com/docs/en/skills）。`.agents/skills/` 是 Cursor 和 OpenClaw 支持的路径。

### Q: OpenClaw 的 skill 在终端和 Dashboard 中是否一样？

A: 完全一样。终端和 Dashboard 共享同一个 Gateway，加载同一套 skills。

### Q: 第三方 skills 包能不能打散放置？

A: 不建议。第三方包内部通常有互相引用的路径（mock-data、Related Skills 等），打散会导致路径断裂。保持原有目录结构，后续更新直接 `git pull`。

### Q: OpenClaw 自定义 skill 不被识别怎么办？

A: 这是已知问题。在 `~/.openclaw/openclaw.json` 中添加 `skills.load.extraDirs` 配置，显式指定 skill 目录路径，然后 `openclaw gateway restart`。

---

## 十一、官方文档链接

| 工具 | 链接 |
|------|------|
| Claude Code Skills | https://code.claude.com/docs/en/skills |
| Cursor Skills | https://cursor.sh/docs/skills |
| OpenClaw Skills | https://docs.openclaw.ai/tools/skills |
| OpenClaw Skills Config | https://docs.openclaw.ai/tools/skills-config |
| OpenClaw Creating Skills | https://docs.openclaw.ai/tools/creating-skills |
