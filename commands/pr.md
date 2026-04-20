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

- **Check for a PR template** in `.github/`, the repo root, or `docs/` (case-insensitive `PULL_REQUEST_TEMPLATE.md`). If `.github/PULL_REQUEST_TEMPLATE/` has multiple templates, ask the user which to use.
- **If a template exists, follow it exactly** — use its sections and fill them in from the diff and commit history. Otherwise, write Context, Acceptance Criteria, and Testing Steps. **Keep the description short and easy to skim**: the diff already shows *what* changed, so focus on *why* and anything non-obvious. Skip file-by-file narration, skip restating the code in prose, and prefer short bullets over paragraphs. When a section has nothing meaningful to add, write a brief note rather than padding with filler.
- **Title**: `TICKET_NUMBER: Brief description` — must be under 72 characters.
- Do NOT include unrelated PR references or auto-linked issue numbers.
- Use `gh pr create --base <confirmed_base_branch>` with HEREDOC for the body.

### 6. Report

Report the PR URL.

### 7. Evaluate Learnings

Evaluate whether this session produced extractable learnings, architectural decisions, or conventions worth preserving. Use whatever knowledge and memory tools are available in this project to save them.
