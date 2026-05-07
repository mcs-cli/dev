---
description: Create Pull Request
model: sonnet
---

# Create Pull Request

Git-only workflow: stage, commit, push, open PR. Never build or test.

Arguments: $ARGUMENTS (optional, e.g. `target develop`, `skip commit`).

## 1. Analyze

Run in parallel: `git status` (no `-uall`), `git diff` (staged + unstaged), `git log origin/HEAD..HEAD --oneline` (fall back to `origin/main` if needed).

Extract the **ticket** from the branch (`__BRANCH_PREFIX__/{ticket}-*` or `{ticket}-*`) or commits. Ask if missing.

## 2. Search project knowledge

After analyzing, search available memory/knowledge tools using keywords from the branch and diff. Look for anything that should shape the PR: PR conventions and templates, CI quirks or required checks, review checklists, prior decisions in the touched modules, known gotchas, related past PRs/issues. Fold relevant findings into the body.

## 3. Commit

- Stage specific files (never `-A`, never `.env` or credentials).
- Re-check `git diff --staged` before writing the message — describe **only** what's actually staged.
- Use the conversation for the *why*, not the *what*.
- Message: one-line summary + up to 3 bullets. HEREDOC.
- Nothing staged → skip.

## 4. Push and pick base branch

In parallel:

**Push** with `-u` if needed.

**Detect base**: default branch from `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'`. For each remote branch (`git branch -r --sort=-committerdate`, excluding the current branch and `HEAD`), compute distance from HEAD to the merge-base via `git rev-list --count <merge-base>..HEAD`. Smallest distance wins; strip `origin/`. Fall back to default on failure.

If the detected base **is not** the default, confirm with `AskUserQuestion`. Otherwise use it. Pass `--base <branch>` to `gh pr create`.

## 5. Create the PR

**Title**: `TICKET: brief description`, under 72 chars.

**Body**: be concise. Write for a reviewer with zero context. Match length to the complexity of the change, not the size of the diff. A few sentences is usually right.

Judgment rules a "be concise" instruction can't infer:

- Lead with **why** — symptom or goal. Don't restate the title.
- Describe **behavior**, not implementation. No file/class/method/line names in the body — the diff shows those.
- Don't report CI-verifiable output (test counts, lint, typecheck, coverage).
- Don't add checkmarks or task-list checkboxes unless the template provides them.
- Don't invent sections ("Design notes", "Out of scope", etc.).
- Don't duplicate between Summary and Changes.
- **Test plan**: imperatives + expected result, e.g. "Run `mcs sync` with a drifted lockfile → expect the migration-hint warning". Numbered if order matters; bullets otherwise. If nothing to verify manually, say so in one line.

**Template**: check `.github/`, repo root, `docs/` for `PULL_REQUEST_TEMPLATE.md` (case-insensitive). If `.github/PULL_REQUEST_TEMPLATE/` has multiples, ask which. Use the template's headings and order; empty sections get `N/A` on one line — do not pad. Keep template-provided checkboxes. No template → use `## Why` · `## Changes` · `## Test plan`.

Create with `gh pr create --base <branch>`, body via HEREDOC.

## 6. Report

Print the PR URL.

## 7. Evaluate learnings

If the session produced reusable knowledge, route it through the available memory/knowledge tools.
