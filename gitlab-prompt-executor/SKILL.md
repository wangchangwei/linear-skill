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

使用 GitLab MCP 工具筛选带 `prompt-done` 标签的 Issue。

GitLab labels 使用 `labels` 参数筛选。

### Step 2: 获取最新评论中的 AI Prompt

使用 GitLab MCP 工具获取 Issue 的评论，找到 AI Task Prompt 内容。

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

使用 GitLab MCP 的 `create_issue_note` 或 `create_note` 提交验证证据到 Issue。

评论内容格式：
```markdown
## 验证证据

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
- Commit: {commitHash}
```

### Step 6: 更新标签

使用 GitLab MCP 工具将标签从 `prompt-done` 改为 `pending-review`。

GitLab API 使用 `labels` 参数更新。

## Quick Reference

| 操作 | 工具 |
|------|------|
| 筛选 prompt-done | GitLab list_issues (labels filter) |
| 获取评论 | GitLab list_issue_notes / get_issue |
| 提交证据 | create_issue_note |
| 更新标签 | update_issue (labels) |
| TDD 执行 | superpowers:test-driven-development |

## GitLab MCP 工具映射

| Linear 工具 | GitLab 工具 |
|-------------|-------------|
| mcp__linear-server__list_issues | gitlab_list_issues (by label) |
| mcp__linear-server__get_issue | gitlab_get_issue |
| mcp__linear-server__list_comments | gitlab_list_issue_notes |
| mcp__linear-server__save_comment | gitlab_create_issue_note |
| mcp__linear-server__save_issue | gitlab_update_issue |

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