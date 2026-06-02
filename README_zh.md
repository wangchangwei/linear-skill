# GitLab Skills

[![English](https://img.shields.io/badge/-English-blue.svg)](README.md)
[![中文](https://img.shields.io/badge/-中文-red.svg)](README_zh.md)

Claude Code 本地 Skills，用于处理 GitLab Issue 和 AI Prompt 的工作流。

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

### GitLab CLI (glab)

**主要工具**：本项目使用 [glab (GitLab CLI)](https://gitlab.com/gitlab-org/cli) 进行所有 GitLab 操作。

#### 安装

**Windows (winget)**：
```bash
winget install GLab.GLab
```

**macOS (Homebrew)**：
```bash
brew install glab
```

**Linux**：
```bash
# Debian/Ubuntu
curl -sSL https://packages.gitlab.com/install/repositories/glab/glab-cli/script.deb.sh | sudo bash
sudo apt-get install glab

# 或使用包管理器
sudo apt install glab  # Debian/Ubuntu
sudo dnf install glab  # Fedora
```

#### 认证

安装后需要认证：

```bash
glab auth login
```

或通过环境变量设置 Token：
```bash
export GITLAB_TOKEN="glpat-xxxxxx"
```

#### 验证安装

```bash
glab --version
glab auth status
```

#### 常用 glab 命令

| 操作 | 命令 |
|------|------|
| 列出 Issues | `glab issue list --label "prompt-done"` |
| 查看 Issue | `glab issue view {issue_id}` |
| 列出评论 | `glab issue note list {issue_id}` |
| 添加评论 | `glab issue note create {issue_id} --message "内容"` |
| 更新标签 | `glab issue update {issue_id} --add-label "pending-review" --remove-label "prompt-done"` |

#### 为什么选择 glab？

- ✅ **标准化**：官方 GitLab CLI 工具
- ✅ **简单**：无需配置 MCP Server
- ✅ **强大**：完整覆盖 GitLab API
- ✅ **可靠**：由 GitLab 团队维护

## 前置配置

### 1. GitLab 标签配置

**必须手动在 GitLab 中创建以下标签**，Skills 依赖这些标签进行状态流转：

| 标签名称 | 用途 | 颜色建议 |
|---------|------|---------|
| `prompt-done` | AI Prompt 已生成，待执行 | 蓝色 |
| `pending-review` | 执行完成，待人工验证 | 黄色 |
| `completed` | 验证通过，任务结束 | 绿色 |

**创建方式**：
1. 登录 GitLab
2. 进入 **Project** → **Settings** → **Labels**
3. 点击 **New label**
4. 输入标签名称（必须精确匹配上述名称）

### 2. GitLab Token 配置

#### 获取 GitLab Token

1. 登录 [GitLab](https://gitlab.com)
2. 进入 **User Settings** → **Access Tokens**
3. 点击 **Add new token**
4. 选择 scopes: `api`, `read_repository`, `write_repository`
5. 复制生成的 Token

#### 配置方式

**方式一：环境变量（推荐）**
```bash
export GITLAB_TOKEN="glpat-xxxxxx"
```

**方式二：glab 配置**
```bash
glab config set token glpat-xxxxxx
```

## Skills

### gitlab-prompt-executor

执行带有 `prompt-done` 标签的 GitLab Issue。

**触发条件**：GitLab Issue 有 `prompt-done` 标签，需要根据评论中的 AI Prompt 执行任务。

**依赖**：
- `superpowers:test-driven-development`
- **glab CLI** (gitlab issue list, view, note create, update)

**工作流程**：
1. 获取带 `prompt-done` 标签的 Issue
2. 从评论中读取 AI Task Prompt
3. 创建开发分支 `{type}/{date}-{short-title}`
4. TDD 循环执行（RED → GREEN → REFACTOR）
5. **E2E 测试 100% 通过后**提交验证证据到 GitLab
6. 将标签从 `prompt-done` 改为 `pending-review`

**核心规则**：测试未通过 = 不能提交证据

**标签流转**：`prompt-done → Execution complete → pending-review → completed`

---

### gitlab-to-task-prompt

将 GitLab Issue 链接转换为 AI 可执行的任务提示词。

**触发条件**：用户提供 GitLab Issue 链接

**依赖**：
- **glab CLI** (gitlab issue view, note list, note create, issue update)

**工作流程**：
1. 获取 Issue 完整信息
2. 通过对话补全 Issue 的背景、目标、约束
3. 生成 AI 任务提示词
4. 追加评论到 GitLab（可选）
5. 添加 `prompt-done` 标签（可选）

---

## 安装方式

将 Skills 复制到本地 Claude Skills 目录：

```bash
cp -r gitlab-prompt-executor ~/.claude/skills/
cp -r gitlab-to-task-prompt ~/.claude/skills/
```

或者使用符号链接：

```bash
ln -s /path/to/linear-skill/gitlab-prompt-executor ~/.claude/skills/gitlab-prompt-executor
ln -s /path/to/linear-skill/gitlab-to-task-prompt ~/.claude/skills/gitlab-to-task-prompt
```

## 快速开始

### 1. 配置 GitLab Token

```bash
export GITLAB_TOKEN="glpat-xxxxxx"
```

### 2. 验证 glab CLI

```bash
glab --version
glab auth status
# 应显示已认证
```

### 3. 使用 Skills

**方式一：通过 GitLab Issue 链接**
```
给我这个 GitLab Issue 的任务描述：[粘贴链接]
```

**方式二：执行 prompt-done Issue**
```
检查一下有没有 prompt-done 的 Issue 需要执行
```

## 常见问题

**Q: Labels 为空或找不到 `prompt-done`**
A: 必须在 GitLab Project Settings → Labels 中手动创建。Skills 不会自动创建标签。

**Q: glab 命令不可用**
A: 检查 glab 是否正确安装，确认 `GITLAB_TOKEN` 环境变量已配置。

**Q: TDD 测试失败**
A: 这是预期行为。测试失败时需修复代码直到测试通过，**不允许跳过**。

**Q: 如何添加 prompt-done 标签**
A: 在 GitLab Issue 页面右侧 Labels → 添加 `prompt-done` 标签（需先在 Project Settings 中创建）。
