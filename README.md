<div align="center">

# Dev Essentials

### Every change starts with a plan and ends in a reviewable pull request.

[![MCS tech pack](https://img.shields.io/badge/MCS-tech%20pack-6f42c1)](https://github.com/mcs-cli/mcs)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-d97757)](https://docs.anthropic.com/en/docs/claude-code)
![macOS](https://img.shields.io/badge/platform-macOS-111111)
![License](https://img.shields.io/badge/license-MIT-2ea44f)

</div>

Claude Code will happily start editing on `main`, write a commit message that restates the diff, and open a pull request with a file-by-file recap nobody needs to read. This pack changes the defaults: it plans before it edits, commits only what is actually staged, and writes pull requests aimed at a reviewer with zero context.

```text
identifier: dev
requires:   mcs >= 2026.2.28
```

## Install

```bash
brew install mcs-cli/tap/mcs      # 1. install mcs
mcs pack add mcs-cli/dev          # 2. register this pack
mcs sync --global --pack dev      # 3. install globally (~/.claude)
mcs doctor                        # 4. verify everything is healthy
```

**Prerequisites:** macOS, [Homebrew](https://brew.sh), and [Claude Code](https://docs.anthropic.com/en/docs/claude-code). `mcs` installs the remaining dependencies through Homebrew: `gh` for pull request operations, `jq` for the session hook, and Node.js for the skill installer.

Global installation is recommended. The pack's only question is a personal branch-naming convention, so install once and every project gets the same git workflow. A repository that needs a different prefix can run `mcs sync --pack dev` from inside it. Drop `--pack dev` to be asked which of your registered packs to sync.

## How it works

**Nothing here guesses.** Every command reads the actual diff, the actual branch, and the actual template.

1. **Sync** — settings, plugins, commands, and the git and code-style sections of `CLAUDE.local.md` are installed. Nothing runs during a session that wasn't put there at sync time.
2. **Session start** — a hook reports the repository and branch, a warning if the branch is protected, uncommitted and stashed counts, merge conflicts, how far ahead or behind the remote you are (or that no upstream is set), and any open pull request.
3. **Planning** — `/make-plan` switches to plan mode so Claude proposes before it edits, researches the code first, then interviews you about the open decisions — as selectable options where the choices are discrete, as prose where they aren't. The plan it writes is short enough to read in one pass and carries its comment and documentation rules with it, so they still apply once you approve it. `/grill-me` stays available for stress-testing anything that isn't a code plan.
4. **Shipping** — `/commit` stages named files and writes a message from what is staged, nothing else. `/pr` adds the push and the pull request, targeting the repository's default branch unless you name another, and shows you the title and body before anything is created.
5. **Reviewing** — `/review-pr` runs specialized agents over the diff for code quality, tests, error handling, comments, and types. Read-only: it reports findings and changes nothing.

## Configuration

Syncing asks one question: the branch prefix used in branch names.

| Prompt | What it does | Default |
|---|---|---|
| **Branch prefix** | Sets the git branch naming convention, as in `feature/ABC-123-login` | `feature` |

The pack also contributes these settings:

| Setting | Value | Purpose |
|---|---|---|
| `alwaysThinkingEnabled` | `true` | Extended thinking on every response |
| `useAutoModeDuringPlan` | `true` | Prefers shell commands over dedicated file tools while planning |
| `ENABLE_TOOL_SEARCH` | `1` | Enables deferred tool search for MCP servers |
| `attribution.commit` | `""` | No Claude Code attribution in commit messages |
| `attribution.pr` | `""` | No Claude Code attribution in pull request descriptions |

## What's included

| Component | What it does |
|---|---|
| **claude-md-management** (plugin) | Audits and improves `CLAUDE.md` files across repositories |
| **claude-hud** (plugin) | Shows context usage, active tools, running agents, and todo progress |
| **pr-review-toolkit** (plugin) | The specialized review agents behind `/review-pr` |
| **session_start.sh** (hook) | Reports repository and branch, protection warning, uncommitted and stashed counts, conflicts, ahead/behind or missing upstream, and any open PR |
| **/make-plan** (command) | Researches the code, interviews you about the open decisions, and writes a short plan that carries its comment and documentation rules into implementation |
| **/commit** (command) | Stages named files, writes a message describing only what is staged, pushes |
| **/pr** (command) | Commit, push, and open a pull request against the default branch — shown for approval before it is created |
| **/review-pr** (command) | Read-only review across code quality, tests, error handling, comments, types, and simplification |
| **grilling** + **grill-me** (skills) | Interviews you with tough questions to pressure-test a plan before you build it. `/make-plan` drives **grilling**; `/grill-me` is the standalone entry point |
| **test-audit** (skill) | Gates every new test on the behavior it protects, and audits existing tests that restate source, duplicate stronger coverage, or keep test-only seams alive. Adapted from [OpenClaw](#credits) |
| **git.md** (template) | Branch naming, read-only review rules, and commit message format in `CLAUDE.local.md` |
| **code-style.md** (template) | Comment and doc-comment rules in `CLAUDE.local.md` — why not what, public declarations only |
| **config/settings.json** (settings) | Always-on extended thinking, deferred tool search, and no Claude attribution in commits or PRs |
| `*.local.*` (gitignore) | Keeps `CLAUDE.local.md` and other local files out of version control |

`mcs doctor` additionally checks that Homebrew is installed and that the `SessionStart` hook is registered.

> The grilling skills are installed from the [`skills`](https://github.com/mattpocock/skills) registry via `npx` during `mcs sync`, which is why Node.js is a dependency.

## Directory structure

```text
dev/
├── techpack.yaml                  # Manifest — defines all components
├── config/
│   └── settings.json              # Claude Code settings (thinking, env vars)
├── hooks/
│   └── session_start.sh           # Git status + branch protection
├── commands/
│   ├── make-plan.md               # /make-plan slash command
│   ├── commit.md                  # /commit slash command
│   ├── pr.md                      # /pr slash command
│   └── review-pr.md               # /review-pr slash command
├── skills/
│   └── test-audit/                # Test authoring gate + audit workflow
└── templates/
    ├── git.md                     # Branch naming + commit conventions
    └── code-style.md              # Comment + doc-comment rules
```

## You might also like

| Pack | Description |
|---|---|
| [memory](https://github.com/mcs-cli/memory) | Persistent, project-specific memory across sessions. Install it and `/pr` and `/review-pr` fold past decisions and known gotchas into their output; without it those steps are simply skipped |
| [ios](https://github.com/mcs-cli/ios) | Xcode integration, simulator management, and Apple documentation |

## Links

- [MCS](https://github.com/mcs-cli/mcs) — the configuration engine
- [Creating Tech Packs](https://github.com/mcs-cli/mcs/blob/main/docs/creating-tech-packs.md)
- [Tech Pack Schema](https://github.com/mcs-cli/mcs/blob/main/docs/techpack-schema.md)

## Credits

The **test-audit** skill is adapted from the [`test-audit`](https://github.com/openclaw/openclaw/tree/main/.agents/skills/test-audit) skill in [OpenClaw](https://github.com/openclaw/openclaw), copyright (c) 2026 OpenClaw Foundation, used under the [MIT License](https://github.com/openclaw/openclaw/blob/main/LICENSE). This version removes the OpenClaw-specific tooling and conventions so it works in any language or framework.

## License

MIT
