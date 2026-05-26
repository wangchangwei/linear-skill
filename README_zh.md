# Linear Skills

[![English](https://img.shields.io/badge/-English-blue.svg)](README.md)
[![中文](https://img.shields.io/badge/-中文-red.svg)](README_zh.md)

Claude Code 本地 Skills，用于处理 Linear Issue 和 AI Prompt 的工作流。

## 前置要求

### 必须安装的 Skills

| Skill | 来源 | 用途 |
|--------|------|------|
| `superpowers:test-driven-development` | [superpowers plugin](https://github.com/superpowerlabs/superpowers) | TDD 循环执行 |
| `superpowers:verification-before-completion` | [superpowers plugin](https://github.com/superpowerlabs/superpowers) | 完成后验证 |

安装 superpowers plugin：
```bash
claude mcp add superpower -- npx -y @superpowerlabs/superpowers
```

### Linear MCP Server

需要配置 `mcp__linear-server__*` 工具，参考 [linear-server-mcp](https://github.com/your-org/linear-server-mcp)。

配置示例（`~/.claude/settings.json`）：
```json
{
  "mcpServers": {
    "linear-server": {
      "command": "npx",
      "args": ["-y", "linear-server-mcp"],
      "env": {
        "LINEAR_API_KEY": "your Linear API key"
      }
    }
  }
}
```

## 前置配置

### 1. Linear 标签配置

**必须手动在 Linear 中创建以下标签**，Skills 依赖这些标签进行状态流转：

| 标签名称 | 用途 | 颜色建议 |
|---------|------|---------|
| `prompt-done` | AI Prompt 已生成，待执行 | 蓝色 |
| `pending-review` | 执行完成，待人工验证 | 黄色 |
| `completed` | 验证通过，任务结束 | 绿色 |

**创建方式**：
1. 登录 Linear Web
2. 进入 **Settings** → **Labels**
3. 点击 **Create Label**
4. 输入标签名称（必须精确匹配上述名称）

### 2. Linear API Key 配置

#### 获取 Linear API Key

1. 登录 [Linear Web](https://linear.app)
2. 进入 **Settings** → **API**
3. 点击 **Create API Key**
4. 复制生成的 Key

#### 配置方式

**方式一：环境变量**
```bash
export LINEAR_API_KEY="lin_api_xxxxxx"
```

**方式二：MCP Server 配置**
在启动 MCP Server 时通过 `env.LINEAR_API_KEY` 传入。

**方式三：Claude Code Settings**
在 `~/.claude/settings.json` 的 `mcpServers.linear-server.env` 中配置。

## Skills

### linear-prompt-executor

执行带有 `prompt-done` 标签的 Linear Issue。

**触发条件**：Linear Issue 有 `prompt-done` 标签，需要根据评论中的 AI Prompt 执行任务。

**依赖**：
- `superpowers:test-driven-development`
- `mcp__linear-server__list_issues`
- `mcp__linear-server__list_comments`
- `mcp__linear-server__save_comment`
- `mcp__linear-server__save_issue`

**工作流程**：
1. 获取带 `prompt-done` 标签的 Issue
2. 从评论中读取 AI Task Prompt
3. 创建开发分支 `{type}/{date}-{short-title}`
4. TDD 循环执行（RED → GREEN → REFACTOR）
5. **E2E 测试 100% 通过后**提交验证证据到 Linear
6. 将标签从 `prompt-done` 改为 `pending-review`

**核心规则**：测试未通过 = 不能提交证据

**标签流转**：`prompt-done → Execution complete → pending-review → completed`

---

### linear-to-task-prompt

将 Linear Issue 链接转换为 AI 可执行的任务提示词。

**触发条件**：用户提供 Linear Issue 链接

**依赖**：
- `mcp__linear-server__get_issue`
- `mcp__linear-server__list_comments`
- `mcp__linear-server__save_comment`
- `mcp__linear-server__save_issue`

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
cp -r linear-prompt-executor ~/.claude/skills/
cp -r linear-to-task-prompt ~/.claude/skills/
```

或者使用符号链接：

```bash
ln -s /path/to/linear-skill/linear-prompt-executor ~/.claude/skills/linear-prompt-executor
ln -s /path/to/linear-skill/linear-to-task-prompt ~/.claude/skills/linear-to-task-prompt
```

## 快速开始

### 1. 配置 Linear API Key

```bash
export LINEAR_API_KEY="lin_api_xxxxxx"
```

### 2. 验证 Linear MCP Server

```bash
claude mcp list
# 应看到 linear-server
```

### 3. 使用 Skills

**方式一：通过 Linear Issue 链接**
```
给我这个 Linear Issue 的任务描述：[粘贴链接]
```

**方式二：执行 prompt-done Issue**
```
检查一下有没有 prompt-done 的 Issue 需要执行
```

## 常见问题

**Q: Labels 为空或找不到 `prompt-done`**
A: 必须在 Linear Settings → Labels 中手动创建。Skills 不会自动创建标签。

**Q: MCP 工具不可用**
A: 检查 `LINEAR_API_KEY` 环境变量是否正确配置，确认 MCP Server 已启动。

**Q: TDD 测试失败**
A: 这是预期行为。测试失败时需修复代码直到测试通过，**不允许跳过**。

**Q: 如何添加 prompt-done 标签**
A: 在 Linear Issue 页面右侧 Labels → 添加 `prompt-done` 标签（需先在 Settings 中创建）。