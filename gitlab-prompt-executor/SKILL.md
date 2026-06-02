---
name: gitlab-prompt-executor
description: Use when there are GitLab issues with prompt-done label that need to be executed via test-driven-development and verified with evidence
---

# GitLab Prompt Executor

## Overview

执行带有 `prompt-done` 标签的 GitLab Issue：从评论中读取 AI Prompt，通过 TDD 方式执行开发/修复，**必须 E2E 测试通过才能提交证据**，提交验证证据到 GitLab，并更新标签为pending-review。

## When to Use

- GitLab Issue 有 `prompt-done` 标签
- 需要根据评论中的 AI Prompt 执行任务
- 任务需要通过 E2E 测试验证

## GitLab CLI 配置

使用前需配置 GitLab Token：

```bash
# 方式一：环境变量
export GITLAB_TOKEN="YOUR_GITLAB_TOKEN"

# 方式二：glab 配置
glab config set token YOUR_GITLAB_TOKEN

# 验证安装
glab auth status
```

## Critical Rule

**E2E 测试必须 100% 通过才能提交证据。未通过测试绝不能提交证据。**

如果测试失败：
1. 记录失败原因
2. 修复代码
3. 重新运行测试
4. 重复直到测试通过
5. 才能提交证据

## Workflow

### Step 1: 获取 prompt-done Issues

使用 **glab CLI** 筛选带 `prompt-done` 标签的 Issue：

```bash
glab issue list --label "prompt-done" --state opened
```

### Step 2: 获取最新评论中的 AI Prompt

使用 **glab CLI** 获取 Issue 的评论，找到 AI Task Prompt 内容：

```bash
glab issue note list {issue_id}
```

### Step 3: 创建开发分支

根据当前所在 git 分支，创建新分支：

```
{type}/{date}-{short-title}
```

- `type`: fix（修复）/ feat（功能）
- `date`: YYYY-MM-DD 格式
- `short-title`: Issue 标题简化（去掉 AI/Agent/支持 等词，保留核心关键词，最多 5 个词）

**分支命名示例**：
- `fix/2026-05-25-oneclick-mode`
- `feat/2026-05-25-user-dashboard`

使用 `git checkout -b {branch_name}` 创建分支。

### Step 4: TDD 执行

使用 `superpowers:test-driven-development` 技能，通过以下循环执行：

```
RED: 编写失败的 E2E 测试
GREEN: 实现功能/修复 bug
REFACTOR: 优化代码
```

**关键要求**：
- 每个 Requirements 必须有对应的测试用例
- 测试必须通过，HTTP 状态码 200 视为成功
- 401/403/500 等均视为失败

**严格规则**：测试必须 100% 通过才能进入下一步。未通过 = 继续修复，不允许跳过或提交虚假证据。

### Step 5: 提交证据到 GitLab

**前提条件**：E2E 测试必须 100% 通过。

使用 **glab CLI** 提交验证证据到 Issue：

```bash
glab issue note create {issue_id} --message '## 验证证据

### 测试结果
- [ ] E2E 测试通过
- [ ] HTTP 200 响应正常

### 证据类型（任选）
**网页截图**: {描述截图内容}
**日志输出**: {关键日志}
**API 响应**: {响应摘要}

### 执行信息
- 执行时间: {timestamp}
- 分支: {gitBranchName}
- Commit: {commitHash}'
```

### Step 6: 更新标签

使用 **glab CLI** 将标签从 `prompt-done` 改为 `pending-review`：

```bash
glab issue update {issue_id} --add-label "pending-review" --remove-label "prompt-done"
```

## Quick Reference

| 操作 | glab 命令 |
|------|----------|
| 筛选 prompt-done | `glab issue list --label "prompt-done" --state opened` |
| 获取评论 | `glab issue note list {issue_id}` |
| 提交证据 | `glab issue note create {issue_id} --message "content"` |
| 更新标签 | `glab issue update {issue_id} --add-label "pending-review" --remove-label "prompt-done"` |
| TDD 执行 | `superpowers:test-driven-development` |

## glab CLI 工具映射

| Linear 工具 | glab 命令 |
|-------------|-----------|
| mcp__linear-server__list_issues | `glab issue list --label "prompt-done"` |
| mcp__linear-server__get_issue | `glab issue view {issue_id}` |
| mcp__linear-server__list_comments | `glab issue note list {issue_id}` |
| mcp__linear-server__save_comment | `glab issue note create {issue_id} --message "content"` |
| mcp__linear-server__save_issue | `glab issue update {issue_id}` |

## 标签状态流转

```
prompt-done → Execution complete → pending-review → 已完成
```

## Red Flags - STOP

以下行为严格禁止：

| 行为 | 后果 |
|------|------|
| 测试未通过就提交证据 | 虚假证据，污染 GitLab |
| 跳过测试步骤 | 无法保证代码正确性 |
| 提交环境问题作为失败理由 | 应修复环境，而非报告失败 |

**所有失败都应被视为待修复的问题，直到测试通过才能交付。**
