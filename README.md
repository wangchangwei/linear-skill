# Linear Skills

Claude Code 本地 Skills，用于处理 Linear Issue 和 AI Prompt 的工作流。

## Skills

### linear-prompt-executor

执行带有 `prompt-done` 标签的 Linear Issue。

**触发条件**：Linear Issue 有 `prompt-done` 标签，需要根据评论中的 AI Prompt 执行任务。

**工作流程**：
1. 获取带 `prompt-done` 标签的 Issue
2. 从评论中读取 AI Task Prompt
3. 创建开发分支 `{type}/{date}-{short-title}`
4. TDD 循环执行（RED → GREEN → REFACTOR）
5. **E2E 测试 100% 通过后**提交验证证据到 Linear
6. 将标签从 `prompt-done` 改为 `待验证`

**核心规则**：测试未通过 = 不能提交证据

**标签流转**：`prompt-done → 执行中 → 待验证 → 已完成`

---

### linear-to-task-prompt

将 Linear Issue 链接转换为 AI 可执行的任务提示词。

**触发条件**：用户提供 Linear Issue 链接

**工作流程**：
1. 获取 Issue 完整信息
2. 通过对话补全 Issue 的背景、目标、约束
3. 生成 AI 任务提示词
4. 追加评论到 Linear（可选）
5. 添加 `prompt-done` 标签（可选）

---

## 安装方式

将 Skills 复制到本地 Claude Skills 目录：

```bash
# linear-prompt-executor
cp -r linear-prompt-executor ~/.claude/skills/

# linear-to-task-prompt
cp -r linear-to-task-prompt ~/.claude/skills/
```

或者使用符号链接：

```bash
ln -s /path/to/linear-skill/linear-prompt-executor ~/.claude/skills/
ln -s /path/to/linear-skill/linear-to-task-prompt ~/.claude/skills/
```

## 前置要求

- Claude Code 环境
- Linear MCP Server (`mcp__linear-server__*` 工具)
- 对于 `linear-prompt-executor`：项目需有 E2E 测试能力

## 依赖工具

| 操作 | 工具 |
|------|------|
| 获取 Issue | `mcp__linear-server__get_issue` |
| 筛选 Issues | `mcp__linear-server__list_issues` |
| 获取评论 | `mcp__linear-server__list_comments` |
| 追加评论 | `mcp__linear-server__save_comment` |
| 更新 Issue | `mcp__linear-server__save_issue` |
| TDD 执行 | `superpowers:test-driven-development` |