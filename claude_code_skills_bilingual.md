# Extend Claude with Skills / 使用 Skills 扩展 Claude 的能力

> **原文来源 / Source:** https://code.claude.com/docs/en/skills

---

## 概述 / Overview

Skills 扩展了 Claude 的能力。创建一个包含指令的 `SKILL.md` 文件，Claude 便会将其纳入工具集。当 Skill 与当前任务相关时，Claude 会自动使用它；你也可以通过 `/skill-name` 直接调用。

当你反复将相同的指令、清单或多步骤操作粘贴到对话中，或 `CLAUDE.md` 中某个章节已演变成操作流程而非事实描述时，就应该创建一个 Skill。与 `CLAUDE.md` 的内容不同，Skill 的内容仅在被调用时加载，因此大量参考资料在不使用时几乎不占用任何 Token。

Claude Code 的 Skills 遵循 [Agent Skills](https://agentskills.io/) 开放标准，可跨多种 AI 工具使用。Claude Code 在此标准基础上扩展了调用控制、子 Agent 执行和动态上下文注入等高级特性。

---

## Bundled Skills / 内置 Skills

Claude Code 内置了一套在每个会话中均可使用的 Skills，包括 `/simplify`、`/batch`、`/debug`、`/loop` 和 `/claude-api`。与大多数直接执行固定逻辑的内置命令不同，内置 Skills 基于 Prompt 驱动：它们向 Claude 提供详细指令，并让 Claude 使用其工具来编排工作。调用方式与其他任何 Skill 相同，即输入 `/` 后跟技能名称。

---

## Getting Started / 快速入门

### Create Your First Skill / 创建第一个 Skill

下面的示例创建一个 Skill，用于汇总 Git 仓库中未提交的变更并标记潜在风险。在 Claude 读取内容之前，它会将实时 diff 注入 Prompt，使响应基于实际工作树，而非 Claude 对已打开文件的推测。当你询问变更内容时，Claude 会自动加载该 Skill；你也可以通过 `/summarize-changes` 直接调用。

**Step 1 — Create the skill directory / 第一步：创建 Skill 目录**

```bash
mkdir -p ~/.claude/skills/summarize-changes
```

注意：当前创建方式为 **个人级 Skills**，个人级 Skills 可在你所有的项目中使用。

**Step 2 — Write SKILL.md / 第二步：编写 SKILL.md**


每个 Skill 都需要一个 `SKILL.md` 文件，由两部分组成：`---` 标记之间的 YAML frontmatter（告知 Claude 何时使用该 Skill），以及 Claude 执行 Skill 时遵循的 Markdown 指令内容。目录名即为你输入的命令名称，`description` 字段帮助 Claude 决定何时自动加载该 Skill。

将以下内容保存至 `~/.claude/skills/summarize-changes/SKILL.md`：

```yaml
---
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
---

## Current changes

!`git diff HEAD`

## Instructions

Summarize the changes above in two or three bullet points, then list any risks you notice such as missing error handling, hardcoded values, or tests that need updating. If the diff is empty, say there are no uncommitted changes.
```

`` !`git diff HEAD` `` 这一行使用了**动态上下文注入**特性：Claude Code 会先执行该命令，并在 Claude 看到 Skill 内容之前，将该行替换为命令输出，从而使指令中内联了当前的 diff。

**Step 3 — Test the skill / 第三步：测试 Skill**

打开一个 Git 项目，对任意文件做一处小修改，然后运行 `claude` 启动 Claude Code。你可以通过以下两种方式测试该 Skill：

```text
# 让 Claude 自动调用 / Let Claude invoke it automatically
What did I change?

# 直接调用 / Or invoke it directly
/summarize-changes
```

---

## Where Skills Live / Skills 的存储位置

Skill 的存储位置决定了其可用范围：

| Location / 位置 | Path / 路径 | Applies to / 适用范围 |
|---|---|---|
| Enterprise / 企业级 | 见 managed settings | 组织内所有用户 |
| Personal / 个人级 | `~/.claude/skills/<name>/SKILL.md` | 你的所有项目 |
| Project / 项目级 | `.claude/skills/<name>/SKILL.md` | 仅当前项目 |
| Plugin / 插件级 | `<plugin>/skills/<name>/SKILL.md` | 启用该插件的范围 |

当不同层级存在同名 Skill 时，企业级覆盖个人级，个人级覆盖项目级。插件 Skills 使用 `plugin-name:skill-name` 命名空间，因此不会与其他层级发生冲突。

### Live Change Detection / 实时变更检测

Claude Code 会监听 Skill 目录中的文件变化。在 `~/.claude/skills/` 或项目 `.claude/skills/` 下新增、编辑或删除 Skill，无需重启即可在当前会话中立即生效。

### Automatic Discovery from Nested Directories / 嵌套目录中的自动发现

当你在子目录中工作时，Claude Code 会自动从嵌套的 `.claude/skills/` 目录中发现 Skills，这适用于各包有各自 Skills 的 Monorepo 项目结构。

### Skill Directory Structure / Skill 目录结构

每个 Skill 都是一个以 `SKILL.md` 为入口的目录：

```text
my-skill/
├── SKILL.md           # Main instructions (required) / 主指令文件（必须）
├── template.md        # Template for Claude to fill in / Claude 填写的模板
├── examples/
│   └── sample.md      # Example output / 输出示例
└── scripts/
    └── validate.sh    # Script Claude can execute / Claude 可执行的脚本
```

---

## Configure Skills / 配置 Skills

### Types of Skill Content / Skill 内容类型

**Reference content / 参考内容**：添加 Claude 在当前工作中应用的知识，如规范、模式、风格指南、领域知识。该内容以内联方式运行，Claude 可在对话上下文中使用。

```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

**Task content / 任务内容**：向 Claude 提供特定操作（如部署、提交、代码生成）的分步指令。这类内容通常需要通过 `/skill-name` 直接调用，而非让 Claude 自行决定。添加 `disable-model-invocation: true` 可防止 Claude 自动触发。

```yaml
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
```

保持 Skill 正文简洁。一旦 Skill 加载，其内容将在整个会话中持续存在于上下文中，因此每一行都会带来持续的 Token 消耗。

---

### Frontmatter Reference / Frontmatter 字段说明

所有字段均为可选。建议至少填写 `description`，以便 Claude 了解何时使用该 Skill。

| Field / 字段 | Required / 必须 | Description / 说明 |
|---|---|---|
| `name` | No | Skill 的显示名称，省略则使用目录名。仅支持小写字母、数字和连字符（最多 64 字符）。 |
| `description` | Recommended | Skill 的功能及使用时机。Claude 据此决定是否应用该 Skill。省略则使用 Markdown 内容的第一段。关键用例请放在最前面，组合文本在 Skill 列表中截断为 1536 字符。 |
| `when_to_use` | No | Claude 调用该 Skill 的额外上下文，如触发短语或示例请求。附加到 `description` 后，计入 1536 字符上限。 |
| `argument-hint` | No | 自动补全时显示的提示，表明期望的参数，例如 `[issue-number]`。 |
| `arguments` | No | 用于 Skill 内容中 `$name` 替换的命名位置参数，接受空格分隔的字符串或 YAML 列表。 |
| `disable-model-invocation` | No | 设为 `true` 可阻止 Claude 自动加载此 Skill，仅允许通过 `/name` 手动触发。默认 `false`。 |
| `user-invocable` | No | 设为 `false` 可从 `/` 菜单中隐藏，适用于不应由用户直接调用的后台知识。默认 `true`。 |
| `allowed-tools` | No | 此 Skill 激活时 Claude 可无需申请权限即可使用的工具列表，接受空格分隔的字符串或 YAML 列表。 |
| `model` | No | 此 Skill 激活时使用的模型。该覆盖仅在当前轮次有效，下一个 Prompt 后恢复会话模型。 |
| `effort` | No | 此 Skill 激活时的努力级别，覆盖会话级别。选项：`low`、`medium`、`high`、`xhigh`、`max`。 |
| `context` | No | 设为 `fork` 可在分叉的子 Agent 上下文中运行。 |
| `agent` | No | `context: fork` 时使用的子 Agent 类型。 |
| `hooks` | No | 限定在此 Skill 生命周期内的 Hooks。 |
| `paths` | No | 限制 Skill 激活的 Glob 模式，仅在匹配文件时 Claude 才会自动加载该 Skill。 |
| `shell` | No | 此 Skill 中 `` !`command` `` 使用的 Shell，接受 `bash`（默认）或 `powershell`。 |

### Available String Substitutions / 可用字符串替换变量

| Variable / 变量 | Description / 说明 |
|---|---|
| `$ARGUMENTS` | 调用 Skill 时传入的所有参数。若内容中不含 `$ARGUMENTS`，参数将以 `ARGUMENTS: ` 形式追加到末尾。 |
| `$ARGUMENTS[N]` | 按 0 为起始索引访问特定参数，如 `$ARGUMENTS[0]` 为第一个参数。 |
| `$N` | `$ARGUMENTS[N]` 的简写，如 `$0` 为第一个参数，`$1` 为第二个。 |
| `$name` | frontmatter `arguments` 列表中声明的命名参数，按顺序对应位置。 |
| `${CLAUDE_SESSION_ID}` | 当前会话 ID，可用于日志记录、创建会话特定文件或关联 Skill 输出与会话。 |
| `${CLAUDE_EFFORT}` | 当前努力级别，可用于根据活跃的努力设置调整 Skill 指令。 |
| `${CLAUDE_SKILL_DIR}` | 包含该 Skill 的 `SKILL.md` 文件的目录路径，可用于引用与 Skill 捆绑的脚本或文件。 |

索引参数使用 Shell 风格的引号。例如，`/my-skill "hello world" second` 会使 `$0` 展开为 `hello world`，`$1` 展开为 `second`。

---

### Add Supporting Files / 添加附属文件

Skills 可在目录中包含多个文件，使 `SKILL.md` 保持简洁，同时让 Claude 仅在需要时访问详细参考资料。

```text
my-skill/
├── SKILL.md          (required / 必须 - overview and navigation / 概述与导航)
├── reference.md      (detailed API docs / 详细 API 文档 - loaded when needed / 按需加载)
├── examples.md       (usage examples / 使用示例 - loaded when needed / 按需加载)
└── scripts/
    └── helper.py     (utility script / 工具脚本 - executed, not loaded / 执行而非加载)
```

在 `SKILL.md` 中引用附属文件，使 Claude 了解各文件的内容及加载时机。保持 `SKILL.md` 在 500 行以内。

```markdown
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

---

### Control Who Invokes a Skill / 控制 Skill 的调用权限

默认情况下，你和 Claude 都可以调用任何 Skill。两个 frontmatter 字段可以限制这一行为：

- **`disable-model-invocation: true`**：仅允许你手动调用。适用于有副作用或需要你控制时机的工作流，如 `/commit`、`/deploy`、`/send-slack-message`——你不会希望 Claude 因为代码看起来已就绪就自行部署。
- **`user-invocable: false`**：仅允许 Claude 自动调用。适用于不应作为命令操作的后台知识，如 `legacy-system-context` Skill 解释旧系统的工作方式，Claude 应在相关时知晓，但 `/legacy-system-context` 对用户来说并无实际意义。

| Frontmatter | You can invoke / 你可调用 | Claude can invoke / Claude 可调用 | When loaded into context / 加载到上下文的时机 |
|---|---|---|---|
| (default / 默认) | Yes | Yes | 描述始终在上下文中，完整内容在调用时加载 |
| `disable-model-invocation: true` | Yes | No | 描述不在上下文中，完整内容在你调用时加载 |
| `user-invocable: false` | No | Yes | 描述始终在上下文中，完整内容在调用时加载 |

---

### Skill Content Lifecycle / Skill 内容生命周期

当你或 Claude 调用一个 Skill 时，渲染后的 `SKILL.md` 内容将作为一条消息进入对话，并在整个会话中保持存在。Claude Code 不会在后续轮次中重新读取 Skill 文件。

自动压缩（Auto-compaction）会在 Token 预算内将已调用的 Skills 保留。当对话被摘要以释放上下文时，Claude Code 会在摘要后重新附加每个 Skill 的最近一次调用内容，保留各自前 5000 个 Token。重新附加的 Skills 共享 25000 个 Token 的总预算。

---

### Pre-approve Tools for a Skill / 为 Skill 预批准工具

`allowed-tools` 字段在 Skill 激活时为列出的工具授予权限，使 Claude 可无需申请审批即可使用它们。它不会限制其他可用工具。

```yaml
---
name: commit
description: Stage and commit the current changes
disable-model-invocation: true
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
```

对于已提交到项目 `.claude/skills/` 目录的 Skills，`allowed-tools` 在你接受工作区信任对话框后生效。在信任仓库之前，请检查项目 Skills，因为 Skill 可能会为自身授予较广泛的工具访问权限。

---

### Pass Arguments to Skills / 向 Skill 传递参数

你和 Claude 都可以在调用 Skill 时传递参数，参数通过 `$ARGUMENTS` 占位符获取。

```yaml
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Understand the requirements
3. Implement the fix
4. Write tests
5. Create a commit
```

运行 `/fix-issue 123` 时，Claude 收到的是"Fix GitHub issue 123 following our coding standards..."

通过位置索引访问单个参数：

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

运行 `/migrate-component SearchBar React Vue` 时，`$0` 替换为 `SearchBar`，`$1` 为 `React`，`$2` 为 `Vue`。

---

## Advanced Patterns / 高级模式

### Inject Dynamic Context / 注入动态上下文

`` !`command` `` 语法在 Skill 内容发送给 Claude 之前执行 Shell 命令，命令输出将替换占位符，使 Claude 接收到实际数据而非命令本身。

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

When this skill runs:
1. Each `` !`...` `` executes immediately (before Claude sees anything) / 每个 `` !`...` `` 立即执行（在 Claude 看到任何内容之前）
2. The output replaces the placeholder in the skill content / 输出替换 Skill 内容中的占位符
3. Claude receives the fully-rendered prompt with actual PR data / Claude 收到含实际 PR 数据的完整渲染 Prompt

这是预处理，不是 Claude 执行的操作，Claude 只看到最终结果。

对于多行命令，使用以 `` ```! `` 开头的围栏代码块代替内联形式。

> **Tip / 提示**：如需在 Skill 运行时触发更深层的推理，可在 Skill 内容中的任意位置包含 `ultrathink`。

---

### Run Skills in a Subagent / 在子 Agent 中运行 Skills

在 frontmatter 中添加 `context: fork` 可让 Skill 在隔离环境中运行。Skill 内容将成为驱动子 Agent 的 Prompt，子 Agent 无法访问你的对话历史。

> **Warning / 注意**：`context: fork` 仅对含有明确指令的 Skills 有意义。若 Skill 内容是"请使用这些 API 规范"这样的指导方针而没有具体任务，子 Agent 接收到指导后无可操作的 Prompt，将直接返回，不产生有意义的输出。

| Approach / 方式 | System prompt | Task / 任务 | Also loads / 同时加载 |
|---|---|---|---|
| Skill with `context: fork` | 来自 Agent 类型（Explore、Plan 等） | SKILL.md 内容 | CLAUDE.md |
| Subagent with `skills` field | 子 Agent 的 Markdown 正文 | Claude 的委托消息 | 预加载 Skills + CLAUDE.md |

**Example: Research skill using Explore agent / 示例：使用 Explore Agent 的研究 Skill**

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

`agent` 字段指定使用哪种子 Agent 配置，选项包括内置 Agent（`Explore`、`Plan`、`general-purpose`）或 `.claude/agents/` 中的任何自定义子 Agent。省略则使用 `general-purpose`。

---

### Restrict Claude's Skill Access / 限制 Claude 对 Skills 的访问

三种控制 Claude 可调用 Skills 的方式：

**1. 禁用所有 Skills / Disable all skills** — 在 `/permissions` 中拒绝 Skill 工具：

```text
# Add to deny rules:
Skill
```

**2. 允许或拒绝特定 Skills / Allow or deny specific skills** — 使用权限规则：

```text
# Allow only specific skills / 仅允许特定 Skills
Skill(commit)
Skill(review-pr *)

# Deny specific skills / 拒绝特定 Skills
Skill(deploy *)
```

Permission syntax: `Skill(name)` for exact match, `Skill(name *)` for prefix match with any arguments.

权限语法：`Skill(name)` 精确匹配，`Skill(name *)` 带任意参数的前缀匹配。

**3. 在 frontmatter 中隐藏单个 Skill / Hide individual skills** — 添加 `disable-model-invocation: true`，将该 Skill 从 Claude 的上下文中完全移除。

---

### Override Skill Visibility from Settings / 通过设置覆盖 Skill 可见性

The `skillOverrides` setting controls skill visibility from your settings instead of the skill's own frontmatter. Use it for skills whose `SKILL.md` you don't want to edit. The `/skills` menu writes it for you: highlight a skill and press `Space` to cycle states, then `Enter` to save to `.claude/settings.local.json`.

`skillOverrides` 设置通过配置文件（而非 Skill 自身的 frontmatter）控制 Skill 可见性，适用于你不想直接编辑 `SKILL.md` 的场景（如共享项目仓库中的 Skills）。`/skills` 菜单可代你写入：高亮一个 Skill，按 `Space` 切换状态，再按 `Enter` 保存到 `.claude/settings.local.json`。

| Value / 值 | Listed to Claude / 对 Claude 可见 | In `/` menu / 在 `/` 菜单中 |
|---|---|---|
| `"on"` | 名称和描述 | 是 |
| `"name-only"` | 仅名称 | 是 |
| `"user-invocable-only"` | 隐藏 | 是 |
| `"off"` | 隐藏 | 隐藏 |

```json
{
  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  }
}
```

Plugin skills are not affected by `skillOverrides`. Manage those through `/plugin` instead.

插件 Skills 不受 `skillOverrides` 影响，请通过 `/plugin` 管理。

---

## Share Skills / 分享 Skills

Skills can be distributed at different scopes depending on your audience:

根据受众的不同，Skills 可以在不同范围内分发：

- **Project skills / 项目 Skills**：将 `.claude/skills/` 提交到版本控制
- **Plugins / 插件**：在你的插件中创建 `skills/` 目录
- **Managed / 托管**：通过 managed settings 在组织范围内部署

---

## Generate Visual Output / 生成可视化输出

Skills 可以捆绑并运行任何语言的脚本，赋予 Claude 超越单次 Prompt 所能实现的能力。一个强大的模式是生成可视化输出——在浏览器中打开的交互式 HTML 文件。

示例：创建一个代码库可视化 Skill，生成可展开折叠的交互式目录树视图。

```bash
mkdir -p ~/.claude/skills/codebase-visualizer/scripts
```

`SKILL.md` uses `${CLAUDE_SKILL_DIR}` to reference the bundled script path, ensuring correct resolution regardless of install level:

`SKILL.md` 使用 `${CLAUDE_SKILL_DIR}` 引用捆绑脚本路径，确保无论安装在哪个层级都能正确解析：

```yaml
---
name: codebase-visualizer
description: Generate an interactive collapsible tree visualization of your codebase.
allowed-tools: Bash(python3 *)
---

Run the visualization script from your project root:

```bash
python3 ${CLAUDE_SKILL_DIR}/scripts/visualize.py .
```

This creates `codebase-map.html` in the current directory and opens it in your browser.
```

要测试，请在任意项目中打开 Claude Code 并询问"Visualize this codebase."，Claude 会运行脚本、生成 `codebase-map.html` 并在浏览器中打开。

该模式适用于任何可视化输出：依赖关系图、测试覆盖率报告、API 文档或数据库 Schema 可视化。

---