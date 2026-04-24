# Create Pull Request

Automate the full commit-push-PR pipeline. This is a **git-only workflow** — never build or test.

Arguments: $ARGUMENTS (optional — additional instructions for this PR, e.g. `target develop` or `skip commit`)

## Steps

### 1. Analyze Changes

- Run `git status` (never use `-uall`) and `git diff` (staged + unstaged) in parallel.
- Run `git log origin/HEAD..HEAD --oneline` to see existing commits on this branch. If `origin/HEAD` is not set, use `git log origin/main..HEAD --oneline` (or the default branch name if different).
- Extract the **ticket number** from the branch name (pattern: `__BRANCH_PREFIX__/{ticket}-*` or `{ticket}-*`) or from commit messages. If not found, ask the user.

### 2. Search Project Knowledge (depends on Step 1 output)

**Do this AFTER analyzing changes** — use keywords from the branch name, changed files, and commit messages to search. Look for:
- PR conventions or templates
- Architectural decisions related to the changed modules
- Context or past issues in the affected areas

Use whatever knowledge and memory tools are available in this project. Include relevant findings in the PR description.

### 3. Stage and Commit

- Stage relevant files (prefer specific files over `git add -A`; never stage `.env` or credentials).
- Describe **what** changed based on the **actual code diff** — the conversation may contain reverted attempts, bugs, or dead ends that don't reflect the final result. Use the conversation for **context and rationale** (the *why*), but never describe changes that aren't in the diff.
- Commit message format: one-line summary + max 3 bullet points describing actual changes. Use HEREDOC for the message.
- If there are no changes to commit, skip to step 4.

### 4. Push and Determine Base Branch

Run these two tasks in parallel:

**Push**: Push the current branch with `-u` flag if needed.

**Determine base branch**:
1. Run `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'` to get the repo's default branch.
2. Get the current branch name with `git branch --show-current`.
3. List candidate remote branches: `git branch -r --sort=-committerdate` — exclude `origin/<current_branch>` and any `HEAD` pointer from the candidates.
4. For each candidate, find the merge-base with HEAD: `git merge-base HEAD <candidate>`. The candidate whose merge-base is the **most recent commit** (i.e., `git rev-list --count <merge-base>..HEAD` returns the smallest number) is the likely parent. Strip the `origin/` prefix to get the base branch name.
5. If detection fails or no candidates are found, fall back to the default branch.
6. If the detected base branch is **not** the default branch, ask the user to confirm using `AskUserQuestion` with options: the detected branch (recommended), the default branch, or Other.
7. If the detected base branch **is** the default branch, use it without asking.
8. Always pass `--base <confirmed_base_branch>` to `gh pr create` in step 5.

### 5. Create the PR

Write for a reviewer who has no context on this branch, issue, or conversation. They should be able to decide **what to focus on** from the body alone. Length should match the complexity of the change, not the size of the diff.

**Title**: `TICKET_NUMBER: Brief description` — under 72 characters. No unrelated PR references or auto-linked issue numbers.

**Body — hard rules:**

- Lead with **why**: the user-visible symptom or goal, and the area of the code affected. Never restate the title.
- Describe **behavior**, not implementation. Do not name files, classes, methods, variables, or line numbers in the body — the diff already shows those. A reviewer should be able to read the PR without knowing the symbol names.
- Bullets are one-line behavior descriptions, not paragraphs. If a bullet needs prose, it's probably duplicating the diff.
- **Do not** report local run results, test counts ("1057 tests pass"), lint/format/typecheck output, coverage numbers, or anything CI already verifies. Reviewers trust CI for that.
- **Do not** add checkmarks (✓, ✅) or GitHub task-list checkboxes (`- [ ]` / `- [x]`) on your own. Use plain bullets — the Test plan is for actions, not a scorecard. Exception: if the template itself provides task-list checkboxes, keep them (see Template handling).
- **Test plan** = steps a reviewer would run to verify the PR works, phrased as imperatives with the expected result (e.g. "Run `mcs sync` with a drifted lockfile → expect the migration-hint warning").
  - Use a **numbered list** when the verification requires steps in a specific order (setup → action → assertion, or multi-step reproductions). Use plain bullets when the verifications are independent.
  - If there is nothing manual to verify, say so in one line.
- **Do not** invent sections ("Design notes", "Implementation notes", "Out of scope", etc.). Rationale belongs in the why; implementation detail belongs in code comments or the diff itself.
- **Do not** duplicate between Summary and Changes. If a bullet restates the summary, delete one.
- If a section has nothing meaningful, write one short line. Under a template, never drop the heading; without a template, omitting is fine.

**Template handling:**

- Check `.github/`, the repo root, and `docs/` for `PULL_REQUEST_TEMPLATE.md` (case-insensitive). If `.github/PULL_REQUEST_TEMPLATE/` has multiple templates, ask the user which to use.
- **If a template exists, use its section headings and order.** Every heading stays and every heading is populated (write "N/A" or a one-line reason if truly empty). The template dictates *structure*, not *length* — fill each section under the hard rules above. If the template provides task-list checkboxes (`- [ ]`), keep them; they're part of the structure the author intended.
- **If no template**, use: `## Why` · `## Changes` · `## Test plan`.

Create with `gh pr create --base <confirmed_base_branch>` using HEREDOC for the body.

### 6. Report

Report the PR URL.

### 7. Evaluate Learnings

Evaluate whether this session produced extractable learnings, architectural decisions, or conventions worth preserving. Use whatever knowledge and memory tools are available in this project to save them.
