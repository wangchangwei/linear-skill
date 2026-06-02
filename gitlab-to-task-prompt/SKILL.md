---
name: gitlab-to-task-prompt
description: Use when user provides a GitLab issue link and wants to dialog for completing the task description and generating an AI-executable prompt
---

# GitLab to Task Prompt

## Overview

将 GitLab Issue 链接转换为完整的 AI 任务提示词。通过对话补全 Issue 信息，生成符合 AI 执行的任务描述。

## When to Use

- 用户提供了一个 GitLab issue 链接
- 需要通过对话补全 Issue 的背景、目标、约束等信息
- 最终生成一个 AI 可以直接执行的任务提示词

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

## Workflow

### Step 1: 提取 Issue 信息

使用 **glab CLI** 获取 Issue 完整信息：

```bash
glab issue view {issue_id}
```

### Step 2: 澄清 Issue 内容（关键步骤）

用户提交的 Issue 通常比较粗糙，**必须先主动提问补全 Issue 本身**，再生成 Prompt。

向用户提问补全以下信息：

| 信息类型 | 问题示例 |
|---------|---------|
| **期望行为** | 正常情况下应该是什么样子？ |
| **实际错误** | 现在的问题具体是什么？错误表现是什么？ |

---

**以下为固定标准（不需询问用户）：**

| 标准 | 规则 |
|------|------|
| **验收标准** | E2E 测试通过即可，但 401/403/500 等非 200 状态码均视为失败 |
| **参考示例** | 项目中 `packages/e2e/tests/` 下的登录相关测试 |
| **路由地址** | 根据功能名称自行检索确定 |

### Step 3: 生成 Prompt

将 Issue 信息 + 对话补全内容组合成 AI 任务提示词：

```markdown
## 任务来源
GitLab Issue: {issue_url}
Issue Title: {title}

## 任务目标
{用户确认的目标}

## 背景信息
{业务背景和技术上下文}

## 约束条件
{技术限制和规范要求}

## 验收标准
{如何验证任务完成}

## 相关资源
{相关代码、文档，设计稿链接}
```

## Prompt 生成模板

```markdown
# Task: {Issue Title}

## Context
{从 GitLab 和对话中获取的背景信息}

## Objective
{清晰的任务目标，一句话描述}

## Requirements
- [ ] {具体需求1}
- [ ] {具体需求2}

## Constraints
{禁止事项和技术限制}

## Verification
{如何验证完成 - 测试、演示、截图等}

## References
- GitLab Issue: {url}
- {其他相关资源}
```

## 常见问题

| 问题 | 解决方案 |
|------|---------|
| Issue 描述不完整 | 通过提问补全：背景、动机、预期结果 |
| 缺少验收标准 | 询问用户：怎么算完成？需要什么证据？ |
| 技术细节模糊 | 询问技术栈、代码位置、相关模块 |
| 涉及多方人员 | 询问 DRI（负责人）和相关协作者 |

## Step 4: 同步到 GitLab（可选）

当用户确认 Prompt 后，同时完成以下两个操作：

1. **添加评论**：使用 **glab CLI** 追加评论
2. **添加标签**：使用 **glab CLI** 添加标签 `prompt-done`

```bash
# 添加评论
glab issue note create {issue_id} --message '## AI Task Prompt

{生成的完整 prompt}

---
*此 Prompt 由 AI 对话生成*'

# 添加标签
glab issue update {issue_id} --add-label "prompt-done"
```

## Quick Reference

| 操作 | glab 命令 |
|------|----------|
| 获取 Issue | `glab issue view {issue_id}` |
| 获取评论 | `glab issue note list {issue_id}` |
| 追加评论 | `glab issue note create {issue_id} --message "content"` |
| 更新 Issue | `glab issue update {issue_id} --add-label "label"` |

## glab CLI 工具映射

| Linear 工具 | glab 命令 |
|-------------|-----------|
| mcp__linear-server__get_issue | `glab issue view {issue_id}` |
| mcp__linear-server__list_comments | `glab issue note list {issue_id}` |
| mcp__linear-server__save_comment | `glab issue note create {issue_id} --message "content"` |
| mcp__linear-server__save_issue | `glab issue update {issue_id}` |
