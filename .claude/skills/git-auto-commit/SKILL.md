---
description: Auto-generate commit message and push to remote. Use when the user says "提交", "commit", "推送", "push", "/git-auto-commit", or asks to commit and push changes.
---

## Git 用户名

!`git config user.name`

## Git 邮箱

!`git config user.email`

## 远程仓库

!`git remote -v`

## 当前变更内容

!`git status`

## 与上次提交的差异

!`git diff HEAD`

## 操作指引

按以下步骤执行：

### 第一步：展示 Git 个人信息

将上面获取到的 git 用户名、邮箱、远程仓库地址展示给用户，格式如下：

```
📋 Git 个人信息确认：
- 用户名: <user.name>
- 邮箱: <user.email>
- 远程仓库: <remote url>
- 当前分支: <branch name>
```

然后询问用户："信息是否正确？确认后我将继续提交。"

**等待用户确认后再继续后续步骤。**

### 第二步：分析变更

根据 `git status` 和 `git diff HEAD` 的输出：
1. 识别所有已修改、新增、删除的文件
2. 理解每个文件变更的目的和内容

### 第三步：生成 Commit Message

根据变更内容自动生成 commit message，遵循以下规则：
- 使用中文撰写
- 第一行为简短摘要（不超过 50 字），格式为：`<类型>: <描述>`
- 类型包括：feat(新功能)、fix(修复)、docs(文档)、refactor(重构)、style(格式)、test(测试)、chore(杂项)
- 如果变更涉及多个方面，可在空行后补充详细说明

### 第四步：执行提交和推送

1. 使用 `git add .` 添加所有变更
2. 使用生成的 commit message 执行 `git commit`
3. 使用 `git push origin <当前分支>` 推送到远程仓库

### 第五步：确认结果

展示提交结果，包括：
- commit hash
- 推送状态（成功/失败）
- 如果推送失败，给出解决建议
