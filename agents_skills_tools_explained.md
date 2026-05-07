# Agents、Skills、Tools 关系详解

> 整理时间：2026-05-07

- [Agents、Skills、Tools 关系详解](#agentsskillstools-关系详解)
  - [一、核心概念一句话总结](#一核心概念一句话总结)
  - [二、用人来类比](#二用人来类比)
  - [三、运行时到底发生了什么](#三运行时到底发生了什么)
  - [四、一个窗口/会话 = 一个 Agent](#四一个窗口会话--一个-agent)
  - [五、单 Agent vs 多 Agents](#五单-agent-vs-多-agents)
  - [六、Skills vs Tools 的区别](#六skills-vs-tools-的区别)
    - [厨师做菜的比喻](#厨师做菜的比喻)
    - [映射到编程场景](#映射到编程场景)
    - [为什么用户容易迷糊](#为什么用户容易迷糊)
  - [七、Cursor 中的主 Agent + subagent](#七cursor-中的主-agent--subagent)
    - [什么是 subagent](#什么是-subagent)
    - [串行 vs 并行（subagent）](#串行-vs-并行subagent)
    - [主 Agent vs subagent 的区别](#主-agent-vs-subagent-的区别)
    - [subagent 的类型（Cursor）](#subagent-的类型cursor)
    - [subagent vs Agents Window](#subagent-vs-agents-window)
  - [八、全景总结图](#八全景总结图)

---

## 一、核心概念一句话总结

| 概念 | 一句话 | 数量关系 |
|------|--------|---------|
| **Agent** | 一个正在为你服务的 AI 对话实例 | 一个窗口 = 一个 Agent |
| **Skill** | 预先写好的操作手册（SKILL.md） | 一个 Agent 可以加载多个 Skills |
| **Tool** | Agent 能调用的实际能力（读写文件、执行命令等） | 一个 Agent 可以使用多个 Tools |
| **Rule** | 始终生效的行为规范 | 一个 Agent 受多条 Rules 约束 |

> **Agent 遵守 Rules，参考 Skills，使用 Tools 来完成你的请求。**

---

## 二、用人来类比

| 概念 | 类比 | 说明 |
|------|------|------|
| **Agent** | 一个员工 | 有能力执行任务的实体，能思考、能调用工具、能对话 |
| **Skill** | 操作手册/SOP | 告诉员工"遇到某种情况时，按这个步骤操作" |
| **Tools** | 员工手里的工具 | 锤子、螺丝刀 → 文件读写、Shell、浏览器等 |
| **Rules** | 公司制度 | 始终生效的行为约束（代码规范、语言偏好等） |

一个员工（Agent）可以掌握多本操作手册（Skills），但操作手册本身不能自己动——必须有员工来执行。

---

## 三、运行时到底发生了什么

```
用户输入消息
      |
      v
Agent 收到消息
      |
      v
Agent 查看已加载的 Skills 列表
（每个 Skill 有触发条件/description）
      |
      v
判断：当前消息匹配哪个 Skill 的触发条件？
      |
  ┌───┴───┐
  匹配      不匹配
  |          |
  v          v
读取该      用通用能力
SKILL.md    回答问题
按指引执行
```

**关键点：** Agent 是主体，Skill 只是被动的文档。Agent 决定是否、何时读取某个 Skill。

---

## 四、一个窗口/会话 = 一个 Agent

| 工具 | 一个窗口/会话 = ? |
|------|-------------------|
| **Cursor Chat** | 1 个 Agent |
| **Claude Code 终端** | 1 个 Agent |
| **OpenClaw Chat**（终端或 Dashboard） | 1 个 Agent |

本质上都一样：**一个对话会话 = 一个 Agent 实例在为你服务。**

OpenClaw 稍有不同：它在 Dashboard "代理" 菜单里可以**配置多个 Agent 角色**（比如 main agent 用 Sonnet，research agent 用 Opus），但每次你对话时，实际处理你消息的还是一个 Agent。

---

## 五、单 Agent vs 多 Agents

| 场景 | 是什么 |
|------|--------|
| 一个 Chat 窗口 | 1 个 Agent 在工作 |
| Cursor Agents Window（多个并行窗口） | 多个 Agent 各自独立运行 |
| Chat 中用 Task 工具派生子任务 | 1 个主 Agent + 若干 subagent（临时工） |

日常说的"Agent"大多数时候就是指**一个 AI 对话实例**。"Agents"（复数）强调的是多个独立实例并行跑。

---

## 六、Skills vs Tools 的区别

### 厨师做菜的比喻

| 概念 | 类比 | 作用 |
|------|------|------|
| **Agent** | 厨师 | 有大脑，能思考、能决策 |
| **Skill** | 菜谱 | 告诉厨师"做宫保鸡丁的步骤是..." |
| **Tool** | 刀、锅、灶台 | 厨师实际用来切菜、炒菜的**物理工具** |

```
Skill（菜谱）写着：
  "第一步：把鸡胸肉切丁"
  "第二步：热锅冷油"
  "第三步：爆炒..."

但是！"切"这个动作需要一把刀（Tool）才能完成。
菜谱本身不能切肉，它只是告诉你应该切。
```

### 映射到编程场景

```
SKILL.md 写着：
  "当用户说'提交代码'时：
   1. 执行 git add .
   2. 生成 commit message
   3. 执行 git commit
   4. 执行 git push"

但是！"执行 git add ." 这个动作需要 Shell 工具（Tool）才能完成。
SKILL.md 本身不能执行命令，它只是告诉 Agent 应该执行什么。
```

完整映射：

```
SKILL.md（操作手册）         Tools（实际能力）
┌─────────────────┐         ┌─────────────────┐
│ 1. 读取文件 xxx  │ ──→需要→│ Read 工具        │
│ 2. 修改第10行    │ ──→需要→│ StrReplace 工具  │
│ 3. 执行测试命令  │ ──→需要→│ Shell 工具       │
│ 4. 提交代码     │ ──→需要→│ Shell 工具       │
└─────────────────┘         └─────────────────┘
      ↑                           ↑
      │                           │
      └──── Agent 读取 ────────────┘
             Agent 调用
```

- **Skill** = 告诉 Agent **做什么**（方向指引）
- **Tool** = 让 Agent **能做到**（执行能力）

没有 Skill → Agent 不知道该做什么（或者得自己想）
没有 Tool → Agent 知道该做什么，但做不到（纸上谈兵）

### 为什么用户容易迷糊

因为 Claude Code / Cursor 内部已经实现了一整套 Tools（Shell、Read、Write、StrReplace 等），这些 Tools 的调用对用户来说是**透明的/隐藏的**。

用户看到的效果是：写了个 SKILL.md → AI 就能自动做事了。

但实际上中间有一步：Agent 读了 SKILL.md 后，**调用内置 Tools** 来完成操作。只是这一步对用户不可见，所以容易觉得"Skill 本身就能做事"。

---

## 七、Cursor 中的主 Agent + subagent

### 什么是 subagent

主 Agent（你对话的那个 AI）有一个工具叫 **Task**，它能派生出一个子 Agent（subagent）去做一件独立的事情，同时主 Agent 可以继续做别的。

### 串行 vs 并行（subagent）

**不用 subagent（串行）：**

```
主 Agent：先重构 utils.py... （等完成）
主 Agent：再去检查 tests/... （等完成）
主 Agent：都好了，报告给你
```

**用 subagent（并行）：**

```
主 Agent：我来重构 utils.py
     同时
     派生 subagent A → "去检查 tests/ 下有没有相关测试"

主 Agent 做自己的活...
subagent A 做自己的活...

subagent A 完成 → 把结果返回给主 Agent
主 Agent 做完 → 综合两边的结果，报告给你
```

### 主 Agent vs subagent 的区别

| | 主 Agent | subagent（临时工） |
|--|---------|-------------------|
| 生命周期 | 整个对话期间一直在 | 做完就消失 |
| 能看到什么 | 你的所有消息 + 对话历史 | 只能看到主 Agent 给它的任务描述 |
| 能做什么 | 所有 Tools | 根据类型，有不同的 Tools 集合 |
| 谁创建的 | 你打开 Chat 时创建 | 主 Agent 调用 Task 工具创建 |
| 结果给谁 | 直接给你 | 返回给主 Agent，由主 Agent 转达给你 |

### subagent 的类型（Cursor）

| 类型 | 用途 | 特点 |
|------|------|------|
| `generalPurpose` | 通用任务 | 能读写文件、执行命令 |
| `explore` | 探索代码库 | 只读，速度快 |
| `shell` | 执行命令 | 专门跑终端命令 |
| `browser-use` | 浏览器操作 | 能打开网页、点击、截图 |

### subagent vs Agents Window

```
┌─────────────────────────────────────────────────────────┐
│ 一个 Chat 内部的 subagent                                │
│                                                          │
│   主 Agent ──派生──→ subagent（做完就没了）                │
│   主 Agent 是老板，subagent 是临时工                      │
│   对你来说：还是一个对话窗口                              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Agents Window（多 Agent 并行）                           │
│                                                          │
│   Agent 1 ──→ 独立做任务 A                               │
│   Agent 2 ──→ 独立做任务 B                               │
│   Agent 3 ──→ 独立做任务 C                               │
│   它们之间没有上下级关系，各干各的                         │
│   对你来说：多个独立窗口同时在跑                          │
└─────────────────────────────────────────────────────────┘
```

简单说：
- **subagent** = 主 Agent 的"手下"，在一个对话内部发生
- **Agents Window** = 多个平级的独立 Agent，各自有独立的对话

---

## 八、全景总结图

```
┌─────────────────────────────────────────────────┐
│                  AI Agent（执行者）               │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Rules    │  │ Skills   │  │ Tools    │      │
│  │(始终生效) │  │(按需触发) │  │(执行能力) │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│                                                  │
│  Agent = Rules + Skills + Tools + LLM 推理能力   │
└─────────────────────────────────────────────────┘
```

- **Rules**：给 Agent 设立的"底线"和"习惯"，每次对话都在
- **Skills**：给 Agent 准备的"按需查阅的手册"，只在触发时加载
- **Tools**：Agent 能调用的"能力"（读写文件、执行命令等）
- **LLM**：Agent 的"大脑"，负责理解和推理

各工具中 Agent 和 Skills 的关系：

| 工具 | Agent 是谁 | Skill 放哪里 | 它们的关系 |
|------|-----------|-------------|-----------|
| **Cursor** | Cursor 内置的 AI Agent | `.cursor/skills/`、`.claude/skills/` 等 | Agent 启动时扫描 Skills，对话时按需触发 |
| **Claude Code** | Claude Code 终端里的 AI Agent | `.claude/skills/` | 同上 |
| **OpenClaw** | Gateway 里配置的 Agent | `~/.openclaw/workspace/skills/` 等 | Gateway 加载 Skills → 注入 Agent 上下文 → Agent 按需使用 |
