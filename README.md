# Linear Skills

[![English](https://img.shields.io/badge/-English-blue.svg)](README.md)
[![中文](https://img.shields.io/badge/-中文-red.svg)](README_zh.md)

Claude Code local Skills for managing Linear Issue and AI Prompt workflows.

## Prerequisites

### Required Skills

| Skill | Source | Purpose |
|-------|--------|---------|
| `superpowers:test-driven-development` | [superpowers plugin](https://github.com/superpowerlabs/superpowers) | TDD execution loop |
| `superpowers:verification-before-completion` | [superpowers plugin](https://github.com/superpowerlabs/superpowers) | Post-completion verification |

Install superpowers plugin:
```bash
claude mcp add superpower -- npx -y @superpowerlabs/superpowers
```

### Linear MCP Server

Requires `mcp__linear-server__*` tools. See [linear-server-mcp](https://github.com/your-org/linear-server-mcp).

Example configuration (`~/.claude/settings.json`):
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

## Setup

### 1. Linear Labels

**You must manually create these labels in Linear** for the Skills to work:

| Label | Purpose | Recommended Color |
|-------|---------|------------------|
| `prompt-done` | AI Prompt generated, awaiting execution | Blue |
| `待验证` | Execution complete, pending verification | Yellow |
| `已完成` | Verified, task complete | Green |

**How to create:**
1. Log in to Linear Web
2. Go to **Settings** → **Labels**
3. Click **Create Label**
4. Enter the exact label name (must match exactly)

### 2. Linear API Key

#### Get Linear API Key

1. Log in to [Linear](https://linear.app)
2. Go to **Settings** → **API**
3. Click **Create API Key**
4. Copy the generated key

#### Configuration Methods

**Option 1: Environment Variable**
```bash
export LINEAR_API_KEY="lin_api_xxxxxx"
```

**Option 2: MCP Server Config**
Pass via `env.LINEAR_API_KEY` when starting MCP Server.

**Option 3: Claude Code Settings**
Configure in `~/.claude/settings.json` under `mcpServers.linear-server.env`.

## Skills

### linear-prompt-executor

Executes Linear Issues tagged with `prompt-done`.

**Trigger**: Issue has `prompt-done` label, requires executing AI Prompt from comments.

**Dependencies**:
- `superpowers:test-driven-development`
- `mcp__linear-server__list_issues`
- `mcp__linear-server__list_comments`
- `mcp__linear-server__save_comment`
- `mcp__linear-server__save_issue`

**Workflow**:
1. Fetch Issues with `prompt-done` label
2. Read AI Task Prompt from comments
3. Create dev branch `{type}/{date}-{short-title}`
4. TDD loop (RED → GREEN → REFACTOR)
5. **Submit verification evidence to Linear** only after E2E tests pass 100%
6. Change label from `prompt-done` to `待验证`

**Core Rule**: No test pass = No evidence submission

**Label Flow**: `prompt-done → 执行中 → 待验证 → 已完成`

---

### linear-to-task-prompt

Converts Linear Issue links to AI-executable task prompts.

**Trigger**: User provides a Linear Issue link

**Dependencies**:
- `mcp__linear-server__get_issue`
- `mcp__linear-server__list_comments`
- `mcp__linear-server__save_comment`
- `mcp__linear-server__save_issue`

**Workflow**:
1. Fetch complete Issue info
2. Dialog to clarify Issue context, goals, and constraints
3. Generate AI task prompt
4. Add comment to Linear (optional)
5. Add `prompt-done` label (optional)

---

## Installation

Copy Skills to local Claude Skills directory:

```bash
cp -r linear-prompt-executor ~/.claude/skills/
cp -r linear-to-task-prompt ~/.claude/skills/
```

Or use symlinks:

```bash
ln -s /path/to/linear-skill/linear-prompt-executor ~/.claude/skills/linear-prompt-executor
ln -s /path/to/linear-skill/linear-to-task-prompt ~/.claude/skills/linear-to-task-prompt
```

## Quick Start

### 1. Configure Linear API Key

```bash
export LINEAR_API_KEY="lin_api_xxxxxx"
```

### 2. Verify Linear MCP Server

```bash
claude mcp list
# Should show linear-server
```

### 3. Use Skills

**Via Linear Issue link**
```
Give me the task description for this Linear Issue: [paste link]
```

**Execute prompt-done Issues**
```
Check if there are any prompt-done Issues to execute
```

## FAQ

**Q: Labels not found or `prompt-done` missing**
A: You must manually create labels in Linear Settings → Labels. Skills do not create labels automatically.

**Q: MCP tools not available**
A: Verify `LINEAR_API_KEY` environment variable is correct and MCP Server is running.

**Q: TDD tests failing**
A: This is expected. Fix the code until tests pass. **Skipping is not allowed.**

**Q: How to add `prompt-done` label**
A: In Linear Issue page, click Labels → add `prompt-done` (must be created in Settings first).