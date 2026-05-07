---
description: Commit and Push
model: haiku
---

# Commit and Push

Git-only workflow: stage, commit, push. No PR. Never build or test.

Branches follow `__BRANCH_PREFIX__/{ticket}-short-description` (e.g. `__BRANCH_PREFIX__/ABC-123-fix-login`).

## 1. Analyze

Run `git status` (no `-uall`) and `git diff` (staged + unstaged) in parallel. Extract the **ticket** from the branch (`__BRANCH_PREFIX__/{ticket}-*` or `{ticket}-*`). Ask if missing.

## 2. Commit

- Stage specific files (never `-A`, never `.env` or credentials).
- Re-check `git diff --staged` before writing the message — describe **only** what's actually staged, ignoring unstaged or stashed changes.
- Use the conversation for the *why*, not the *what*. Don't mention reverted attempts or dead ends.
- Be concise: one-line summary + up to 3 bullets. Each bullet a short behavior description, not prose. HEREDOC.
- Nothing staged → say so and stop.

## 3. Push

Push the current branch with `-u` if needed.
