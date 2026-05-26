# GitLab Skills

[![English](https://img.shields.io/badge/-English-blue.svg)](README.md)
[![中文](https://img.shields.io/badge/-中文-red.svg)](README_zh.md)

Claude Code local Skills for managing GitLab Issue and AI Prompt workflows.

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

### GitLab MCP Server

Requires GitLab MCP tools. See [@zereight/mcp-gitlab](https://github.com/zereight/gitlab-mcp).

Example configuration (`~/.claude/settings.json`):
```json
{
  "mcpServers": {
    "gitlab": {
      "command": "npx",
      "args": ["-y", "@zereight/mcp-gitlab"],
      "env": {
        "GITLAB_PERSONAL_ACCESS_TOKEN": "your GitLab token"
      }
    }
  }
}
```

## Setup

### 1. GitLab Labels

**You must manually create these labels in GitLab** for the Skills to work:

| Label | Purpose | Recommended Color |
|-------|---------|------------------|
| `prompt-done` | AI Prompt generated, awaiting execution | Blue |
| `pending-review` | Execution complete, pending verification | Yellow |
| `completed` | Verified, task complete | Green |

**How to create:**
1. Log in to GitLab
2. Go to **Project** → **Settings** → **Labels**
3. Click **New label**
4. Enter the exact label name (must match exactly)

### 2. GitLab Token

#### Get GitLab Token

1. Log in to [GitLab](https://gitlab.com)
2. Go to **User Settings** → **Access Tokens**
3. Click **Add new token**
4. Select scopes: `api`, `read_repository`, `write_repository`
5. Copy the generated token

#### Configuration Methods

**Option 1: Environment Variable**
```bash
export GITLAB_TOKEN="glpat-xxxxxx"
```

**Option 2: MCP Server Config**
Pass via `env.GITLAB_TOKEN` when starting MCP Server.

**Option 3: Claude Code Settings**
Configure in `~/.claude/settings.json` under `mcpServers.gitlab.env`.

## Skills

### gitlab-prompt-executor

Executes GitLab Issues tagged with `prompt-done`.

**Trigger**: Issue has `prompt-done` label, requires executing AI Prompt from comments.

**Dependencies**:
- `superpowers:test-driven-development`
- GitLab MCP tools (gitlab_get_issue, gitlab_list_issues, etc.)

**Workflow**:
1. Fetch Issues with `prompt-done` label
2. Read AI Task Prompt from comments
3. Create dev branch `{type}/{date}-{short-title}`
4. TDD loop (RED → GREEN → REFACTOR)
5. **Submit verification evidence to GitLab** only after E2E tests pass 100%
6. Change label from `prompt-done` to `pending-review`

**Core Rule**: No test pass = No evidence submission

**Label Flow**: `prompt-done → Execution complete → pending-review → completed`

---

### gitlab-to-task-prompt

Converts GitLab Issue links to AI-executable task prompts.

**Trigger**: User provides a GitLab Issue link

**Dependencies**:
- GitLab MCP tools (gitlab_get_issue, gitlab_list_issue_notes, etc.)

**Workflow**:
1. Fetch complete Issue info
2. Dialog to clarify Issue context, goals, and constraints
3. Generate AI task prompt
4. Add comment to GitLab (optional)
5. Add `prompt-done` label (optional)

---

## Installation

Copy Skills to local Claude Skills directory:

```bash
cp -r gitlab-prompt-executor ~/.claude/skills/
cp -r gitlab-to-task-prompt ~/.claude/skills/
```

Or use symlinks:

```bash
ln -s /path/to/linear-skill/gitlab-prompt-executor ~/.claude/skills/gitlab-prompt-executor
ln -s /path/to/linear-skill/gitlab-to-task-prompt ~/.claude/skills/gitlab-to-task-prompt
```

## Quick Start

### 1. Configure GitLab Token

```bash
export GITLAB_TOKEN="glpat-xxxxxx"
```

### 2. Verify GitLab MCP Server

```bash
claude mcp list
# Should show gitlab server
```

### 3. Use Skills

**Via GitLab Issue link**
```
Give me the task description for this GitLab Issue: [paste link]
```

**Execute prompt-done Issues**
```
Check if there are any prompt-done Issues to execute
```

## FAQ

**Q: Labels not found or `prompt-done` missing**
A: You must manually create labels in GitLab Project Settings → Labels. Skills do not create labels automatically.

**Q: MCP tools not available**
A: Verify `GITLAB_TOKEN` environment variable is correct and MCP Server is running.

**Q: TDD tests failing**
A: This is expected. Fix the code until tests pass. **Skipping is not allowed.**

**Q: How to add `prompt-done` label**
A: In GitLab Issue page, click Labels → add `prompt-done` (must be created in Project Settings first).