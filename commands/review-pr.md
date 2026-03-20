# Comprehensive PR Review

Run a context-aware pull request review using multiple specialized agents, each focusing on a different aspect of code quality. This is a **read-only review** — do NOT make code changes or post GitHub comments unless explicitly asked.

Arguments: $ARGUMENTS (optional — PR number/URL, review aspects, or flags)

## Steps

### 1. Gather PR Context

Determine the PR under review:

1. Parse `$ARGUMENTS` for a PR number (e.g. `123`), a GitHub PR URL, or review aspect keywords (`code`, `simplify`, `tests`, `errors`, `comments`, `types`, `all`).
2. If a PR number or URL is found, run: `gh pr view <number-or-url> --json number,title,body,headRefName,baseRefName,url,state`
3. If no PR identifier is provided, run: `gh pr view --json number,title,body,headRefName,baseRefName,url,state` (uses current branch).
4. If no PR exists for the current branch, treat this as a **pre-PR review**:
   - Determine the base branch by trying in order: `origin/develop`, `origin/main`, `origin/master`.
   - All diffs will be computed against this base.
5. Extract and note: **PR title**, **PR description**, **head branch**, **base branch**, **ticket number** (from branch pattern `*/{ticket}-*` or commit messages), **PR URL** (if exists).

### 2. Ensure Local Branch

Ensure the local workspace has the PR changes:

1. Get current branch: `git rev-parse --abbrev-ref HEAD`
2. If current branch matches the PR head branch → run `git fetch origin` and check if local is behind remote. Pull if needed.
3. If current branch differs from the PR head branch:
   - If the branch exists locally → `git checkout <pr-branch> && git pull`
   - If not → `git fetch origin <pr-branch> && git checkout <pr-branch>`
4. For pre-PR review (no PR exists) → stay on current branch.

### 3. Read the Diff (BLOCKING — must complete before Step 4)

Identify changed files and read the full diff: `git diff <base>...<head>`

### 4. Search Project Knowledge (depends on Step 3 output)

Use keywords extracted from the diff content, changed file names, and module names from Step 3. Look for:
- PR conventions or review standards
- Architectural decisions related to the changed modules
- Past review findings or known issues in the affected areas
- Coding standards specific to the file types being changed

Use whatever knowledge and memory tools are available in this project. Include relevant findings in each agent's context preamble.

### 5. Determine Review Scope

1. Parse `$ARGUMENTS` for review aspect filters:
   - `code` — general code review
   - `simplify` — code simplification
   - `tests` — test coverage analysis
   - `errors` — silent failure detection
   - `comments` — comment accuracy
   - `types` — type design analysis
   - `all` — all applicable reviews (default)
2. **Default behavior** (no aspect filter or `all`):
   - **Always run:** `pr-review-toolkit:code-reviewer`
   - **Run if test files changed or new functionality added:** `pr-review-toolkit:pr-test-analyzer`
   - **Run if error handling / catch blocks / Result types in diff:** `pr-review-toolkit:silent-failure-hunter`
   - **Run if comments/documentation substantially changed:** `pr-review-toolkit:comment-analyzer`
   - **Run if new types/structs/classes/enums/protocols added:** `pr-review-toolkit:type-design-analyzer`
   - **Run after code-reviewer completes (polish pass):** `pr-review-toolkit:code-simplifier`
3. If specific aspects are provided in arguments, run **only** those agents.

### 6. Launch Review Agents

Run **all selected agents in parallel** by default (launch simultaneously). If `--sequential` appears in `$ARGUMENTS`, run agents one at a time instead.

For EACH agent, include this context preamble in the task prompt:

> **PR Context:**
> - Title: {PR title}
> - Description: {PR description summary}
> - Branch: {head} -> {base}
> - Ticket: {ticket number}
> - Changed files: {list}
>
> Focus on the diff between {base} and {head}. This is a read-only review — do NOT make code changes or post GitHub comments.

### 7. Aggregate Results

After all agents complete, produce a unified summary:

```
# PR Review Summary

## PR: {title} (#{number} or "pre-PR review")
Branch: {head} -> {base} | Ticket: {ticket}

## Critical Issues (X found)
- [agent-name]: Issue description [file:line]

## Important Issues (X found)
- [agent-name]: Issue description [file:line]

## Suggestions (X found)
- [agent-name]: Suggestion [file:line]

## Strengths
- What's well-done in this PR

## Recommended Action
1. Fix critical issues first
2. Address important issues
3. Consider suggestions
4. Re-run /review-pr after fixes
```

## Usage Examples

**Review current branch's PR (default — all agents, parallel):**
`/review-pr`

**Review a specific PR:**
`/review-pr 1234`

**Review specific aspects only:**
`/review-pr tests errors`

**Sequential execution:**
`/review-pr --sequential`
