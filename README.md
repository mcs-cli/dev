# Core Pack

A [tech pack](https://github.com/mcs-cli/mcs) that provides foundational settings, plugins, and git workflows for Claude Code.

Built for the [`mcs`](https://github.com/mcs-cli/mcs) configuration engine.

```
identifier: dev
requires:   mcs >= 2026.2.28
```

---

## What Is This?

This pack sets up the baseline Claude Code experience — plan mode by default, extended thinking, structured git workflows, and a curated set of plugins.

On session start, the pack reports git status, branch protection warnings, ahead/behind tracking, and open PRs.

---

## What's Included

### Plugins

| Plugin | Description |
|--------|-------------|
| **explanatory-output-style** | Structured, educational response formatting with insight callouts |
| **ralph-loop** | Iterative refinement loop for complex multi-step tasks |
| **claude-md-management** | Audit and improve `CLAUDE.md` files across repositories |
| **claude-hud** | On-screen display showing context usage, active tools, and agent status |
| **pr-review-toolkit** | Specialized review agents for comprehensive PR analysis |

### Session Hooks

| Hook | Event | What It Does |
|------|-------|-------------|
| **session_start.sh** | `SessionStart` | Injects git status, branch protection warnings, ahead/behind tracking, open PRs |

### Slash Commands

| Command | Description |
|---------|-------------|
| `/commit` | Stage, commit, push — analyzes the actual diff, writes structured commit messages |
| `/pr` | Full pipeline: commit > push > create PR via `gh` with ticket extraction |
| `/review-pr` | Context-aware PR review using specialized agents (code quality, tests, errors, simplification) |

### Templates (CLAUDE.local.md)

| Section | Instructions |
|---------|-------------|
| **git** | Branch naming conventions, read-only PR reviews, commit message format |

### Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| `defaultMode` | `plan` | Claude asks for approval before making changes |
| `alwaysThinkingEnabled` | `true` | Extended thinking on every response |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `claude-sonnet-4-6` | Upgrades lightweight model tasks to Sonnet |
| `ENABLE_TOOL_SEARCH` | `1` | Enables deferred tool search for MCP servers |

---

## Installation

### Prerequisites

- macOS (Apple Silicon or Intel)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

### Setup

```bash
# 1. Install mcs
brew install mcs-cli/tap/mcs

# 2. Register this tech pack
mcs pack add mcs-cli/dev

# 3. Sync your project
cd ~/Developer/my-project
mcs sync

# 4. Verify everything is healthy
mcs doctor
```

During `mcs sync`, you'll be prompted for:

| Prompt | What It Does | Default |
|--------|-------------|---------|
| **Branch prefix** | Sets git branch naming convention (e.g. `feature/ABC-123-login`) | `feature` |

---

## Directory Structure

```
dev/
├── techpack.yaml                  # Manifest — defines all components
├── config/
│   └── settings.json              # Claude Code settings (plan mode, env vars)
├── hooks/
│   └── session_start.sh           # Git status + branch protection
├── commands/
│   ├── commit.md                  # /commit slash command
│   ├── pr.md                      # /pr slash command
│   └── review-pr.md               # /review-pr slash command
└── templates/
    └── git.md                     # Branch naming + commit conventions
```

---

## You Might Also Be Interested In

| Pack | Description |
|------|-------------|
| [memory](https://github.com/mcs-cli/memory) | Persistent memory and knowledge management — gives Claude long-term recall across sessions |
| [ios](https://github.com/mcs-cli/ios) | Xcode integration, simulator management, and Apple documentation |

---

## Links

- [MCS](https://github.com/mcs-cli/mcs) — the configuration engine
- [Creating Tech Packs](https://github.com/mcs-cli/mcs/blob/main/docs/creating-tech-packs.md) — guide for building your own
- [Tech Pack Schema](https://github.com/mcs-cli/mcs/blob/main/docs/techpack-schema.md) — full YAML reference

---

## License

MIT
