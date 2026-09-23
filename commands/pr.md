---
description: Create Pull Request
---

# Create Pull Request

Git-only workflow: stage, commit, push, open PR. Never build or test.

Arguments: $ARGUMENTS (optional) — `target <branch>` / `base <branch>` / `--base <branch>` / a bare branch name to override the base; `skip commit`.

## 1. Analyze

**Base branch.** If `$ARGUMENTS` names one — `target <x>`, `base <x>`, `--base <x>`, or a bare token that isn't a reserved word (`skip`, `commit`) — use it. Otherwise use the repository default:

```
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

If `gh` fails (offline, unauthenticated), fall back to `git symbolic-ref --short refs/remotes/origin/HEAD` and strip `origin/`. **Never scan remote branches. Never ask.**

Then confirm `origin/<base>` resolves with `git rev-parse --verify -q origin/<base>`. **Verify whichever form produced the base, including an explicit argument.** Only if that fails — a narrow fetch refspec or an upstream rename can leave the ref absent locally — run `git fetch origin <base>` and verify again. Fetching costs about two seconds against 30ms for the check, so it is the recovery path, never the default one. If a branch you were *given* still doesn't resolve, stop and say so; never silently fall back to the default when the user named a branch.

Keep the **bare** name — `gh pr create --base` takes `main`, not `origin/main`. Use `origin/<base>` wherever a ref is needed, so a stale local branch is never read.

**If the current branch is the base, stop.** Say so and suggest creating a branch. Do not stage, commit, or push.

Then run in parallel: `git status` (no `-uall`), `git diff` (staged + unstaged), `git log origin/<base>..HEAD --oneline`.

Extract the **ticket** from the branch (`__BRANCH_PREFIX__/{ticket}-*` or `{ticket}-*`) or commits. Ask if missing.

## 2. Search project knowledge

After analyzing, search available memory/knowledge tools using keywords from the branch and diff. Look for anything that should shape the PR: PR conventions and templates, CI quirks or required checks, review checklists, prior decisions in the touched modules, known gotchas, related past PRs/issues. Fold relevant findings into the body.

## 3. Commit

- Stage specific files (never `-A`, never `.env` or credentials).
- Re-check `git diff --staged` before writing the message — describe **only** what's actually staged.
- Use the conversation for the *why*, not the *what*.
- Message: one-line summary + up to 3 bullets. HEREDOC.
- Nothing staged → skip.

## 4. Push

Push the current branch with `-u` if needed.

## 5. Draft the PR

**Title**: `TICKET: brief description`, under 72 chars.

**Budget.** Whole body under 150 words. A one-line rename or config bump is a one-line PR. Only a template's own sections justify going longer.

**Shape.** Prose for the why — two or three plain sentences read faster than dashed fragments.

Rules:

- Lead with **why** — symptom, goal, or constraint. Don't restate the title.
- Describe **behavior**, not implementation. No file, class, or method names; the diff shows those.
- If a sentence restates the diff, the title, or the branch, delete it. That includes `(unchanged)` notes and `Ticket:` / `Branch:` / `JIRA:` metadata lines.
- Don't report CI-verifiable output — test counts, lint, typecheck, coverage.
- Don't invent sections, and don't add checkboxes the template didn't provide.

**Test plan**: bullets, at most five, each an imperative + expected result — "Run `mcs sync` with a drifted lockfile → expect the migration-hint warning". Number them instead if order matters. If nothing to verify manually, say so in one line — don't omit the section, but don't pad it either.

**Template**: check `.github/`, repo root, and `docs/` for `PULL_REQUEST_TEMPLATE.md` (case-insensitive). If `.github/PULL_REQUEST_TEMPLATE/` has multiples, ask which. Use its headings and order — **all** of them, including a `## Changes` if the template provides one — and apply the budget within each section. Keep template-provided checkboxes. Empty sections get `N/A` on one line.

No template → `## Why` + `## Test plan`, and nothing else unless the change carries something the diff genuinely can't show (a migration step, a behavior toggle, non-obvious sequencing).

## 6. Review and create

Ask with `AskUserQuestion`: **Create** / **Revise** / **Cancel**, with the title and full body as the `preview` of **Create** (listed first). The preview is the only place the draft is guaranteed to be seen — never ask without it. On *Revise*, apply the feedback and ask again with the new draft.

On approval, create with `gh pr create --base <base>`, body via HEREDOC. Print the PR URL.

## 7. Evaluate learnings

If the session produced reusable knowledge, route it through the available memory/knowledge tools.
