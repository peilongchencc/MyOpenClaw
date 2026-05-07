# MyOpenClaw

开始养虾🦐

本仓库记录了笔者学习 Claude Code 和 OpenClaw 的完整笔记。

## 笔者电脑配置

| 配置项 | 版本信息 |
| --- | --- |
| 操作系统 | macOS 26.3.1 (a) |
| OpenClaw版本 | OpenClaw 2026.4.26 |
| 系统级Node版本 | v24.15.0 (Homebrew node@24) |
| NVM的Node版本 | v24.11.1 |

## 新手上手建议

1. 先跑通 `openclaw gateway status`，确认服务正常
2. 访问 Dashboard http://127.0.0.1:18789 看看界面，熟悉功能
3. 先装安全技能 Skill-Vetter，再按需安装其他 Skills
4. 配置 USER.md，告诉 Agent 你的技术背景（比如 Python/FastAPI 后端开发），让 AI 更懂你
5. 先单 Agent 用熟，再考虑多 Agent 协作

在 `~/.openclaw/openclaw.json` 中可以配置多个 Agent，并分配不同模型。

角色分工建议（混合模型省 40-60% 成本）：

- 路由/编排：Claude Haiku 4.5（便宜，响应约 200ms）
- 代码：Claude Sonnet 4.6 / Gemini 3.1 Pro
- 长文写作：DeepSeek V4 Pro（支持 1M Token 上下文）
- 研究综合：GPT-5.4 / Claude Opus 4.6

## 文档导航

| 文档 | 内容 | 适合谁看 |
|------|------|----------|
| [openclaw_guide.md](openclaw_guide.md) | OpenClaw 完整使用指南（架构、配置、运维、性能优化） | OpenClaw 用户 |
| [claude_code_guide.md](claude_code_guide.md) | Claude Code 安装配置指南（接入千问模型） | Claude Code 用户 |
| [skills_learning_notes.md](skills_learning_notes.md) | Skills 概念、原理、演化、趋势 | 想理解 Skills 底层机制的人 |
| [skills_practical_guide.md](skills_practical_guide.md) | Skills 实战：创建、配置、验证（覆盖三个工具） | 想动手写 Skill 的人 |
| [openclaw_skills_bilingual.md](openclaw_skills_bilingual.md) | OpenClaw Skills 官方文档（中英对照） | 需要查阅 OpenClaw 官方参考 |
| [claude_code_skills_bilingual.md](claude_code_skills_bilingual.md) | Claude Code Skills 官方文档（中英对照） | 需要查阅 Claude Code 官方参考 |
