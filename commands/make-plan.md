---
description: Research, grill, and write a plan
---

# Make Plan

Turn an objective into a plan worth approving. Research first, settle the open decisions with the user, then write something short enough to read in one pass.

Arguments: $ARGUMENTS — the objective. Prefix with `quick` to skip the grilling round.

## 1. Frame

**Call `EnterPlanMode` first**, unless already in plan mode. Nothing here touches project files — the mode is what guarantees it. The single exception is the plan file itself, written in Step 5.

Plan mode brings its own workflow. Where it and this command disagree on process, this command wins; the plan file's location is the harness's.

Then read `$ARGUMENTS`. If its first whitespace-separated token is exactly `quick`, set **quick mode** and drop that token; `quickly refactor the parser` is an objective, not a flag. Restate what remains in one line.

**If the session already discussed this objective, that discussion is an input.** Often the command is invoked as "based on our discussion, plan this" — carry forward what it established: options ruled out, constraints agreed, decisions made. Research what it left open, not what it settled.

## 2. Search project knowledge

Search available memory/knowledge tools using keywords from the objective and the modules it touches. Look for prior decisions, known gotchas, conventions that constrain the approach, and past attempts at the same problem. A plan that contradicts a settled decision is worse than no plan.

## 3. Research

Read the actual code before proposing anything. Trace the real flow end to end, through every file the change touches.

**Facts are your job, decisions are the user's.** Never ask what you can look up. Dispatch sub-agents for broad searches; ask the user only what the codebase can't answer.

If the code contradicts the objective's premise, stop before Step 4 — don't plan around it. Put the finding to the user with `AskUserQuestion`: what you found, and the real options, including doing nothing. Continue only on their answer. A well-built plan for the wrong problem is the expensive failure here.

## 4. Grill

Skip entirely in **quick mode**.

Otherwise call the Skill tool with `grilling` and follow it — the design tree, the frontier, the rounds — with one override on **how** questions reach the user:

- **Discrete choice → `AskUserQuestion`.** Your recommendation is option 1, its label suffixed `(Recommended)`. Cap is 4 questions per call: a wider frontier takes a second call, never a trimmed frontier.
- **Genuinely open → prose**, in the skill's own format. A question with no enumerable answers isn't improved by inventing three.

Batch the frontier. One question at a time is the slow path the rounds exist to avoid. A decision the earlier conversation already settled is not on the frontier — don't re-ask it.

A round with both kinds: print the prose questions and wait for those answers first, then ask the discrete ones with `AskUserQuestion`. The dialog can hide text printed before it, so never send both in one message.

The skill's closing confirmation is the plan approval in Step 8. When the frontier is empty, go to Step 5 — don't ask whether you've reached a shared understanding.

`AskUserQuestion`'s usual bar — only what you can't settle from sensible defaults — doesn't apply here: the user ran this command to be asked, so a decision you *could* default is still theirs.

## 5. Write the plan

Write it where the harness expects it — plan mode names a plan file; write there. It stays a draft until Step 7 passes.

**Concise by construction.** The per-section caps below are the budget; there's no global word count to game. A rename is a three-line plan, a twenty-file refactor is twenty one-line steps. A plan may run long because it has many steps — never because a section grew.

**Readable.** Write for someone deciding in a few minutes. Plain sentences, no throat-clearing, no paragraph whose real job is to look thorough. If a line could be cut without changing the decision, cut it.

**Self-contained.** A new session with none of this conversation must be able to execute the plan from the file alone. So:

- Locate code by file and symbol (function, type, key, heading), never by line number. Line numbers go stale with the first edit.
- Each step says what changes concretely enough to do without guessing: the new value, the renamed symbol, the behavior added or removed.
- No pointers into the conversation: no "as discussed", "option B", "the approach above", "the issue we found". Say the thing itself.
- Any term the code doesn't define gets defined in the plan, in one clause.

**Shape:**

- **Why** — two sentences. The symptom, goal, or constraint. Don't restate the objective back at the user.
- **Steps** — numbered, one line each, naming the file and what changes in it. No paragraph under a step arguing the step is a good idea. Group them under phase headings **only** when something happens between the groups — a review, a deploy, a separate PR, a checkpoint where you'd stop and verify before continuing. Work that runs straight through is one ungrouped list, however long.
- **Verify** — how the user confirms it worked. Bullets, at most four, each a check plus its expected result. One line saying so if there's nothing to check manually.
- **Risks** — only when a step can fail in a way the user would want to hear about first. Omit the heading otherwise.

**No research narration.** What you learned while researching belongs in the conversation or a memory file, not the plan. Where a `Context` heading is required — some plan-mode harnesses mandate one — the two-sentence Why goes under it and nothing else. This is the single biggest source of plan bloat.

Cut every sentence arguing *for* a decision already settled. The decision itself belongs in Step 6 as one line; the case for it belongs nowhere. A plan is what you'll do, not the argument that you should.

## 6. Carry the constraints

End every plan with this block verbatim, plus anything settled with the user — in the grilling round or earlier in the conversation. One line each, no rationale: the plan file outlives the conversation, so a decision that lives only in chat is lost.

```
Constraints:
- Work on a branch, never the default branch.
- If the code disagrees with a step, stop and ask before deviating from it.
- If the steps are grouped into phases, stop after each phase and wait for the user.
- When the steps are done, run every Verify check you can and report each result; list the ones only the user can run.
- Comments explain why, never what. No comment that restates the line below it.
- Doc comments on public declarations only.
- Match the file's existing comment density.
```

Emit it even when the project or user instructions already say the same — where they do, those govern in full. Often they won't. These travel *with* the plan so they're re-read at implementation time, including by a session that never saw this command: honor them while building, not just while writing.

## 7. Check the draft

Two passes, both before the user sees anything. Every fix goes into the plan file itself — the user is shown the corrected plan, never a draft with a list of errata.

**Facts.** List every factual claim in the draft: file paths, symbol names and locations, counts, and statements about current behavior. Verify each one with a tool call made **after** the draft was written. Reads from Step 3 don't count, and counts must come from an actual count.

Print the table in chat:

| Claim | Checked with | Result |
|---|---|---|
| … | `Grep …` / `Read …:L` | ✅ / ❌ fixed to … / ⚠️ dropped |

- ❌ → fix the plan, or drop the step if the fix isn't possible.
- If a ❌ undercuts a decision the user made, ask them with `AskUserQuestion` before presenting.
- Settled decisions are not facts to re-check. This pass verifies the plan, it doesn't reopen it.
- **Any edit to the plan file, including after a rejection, needs fresh rows for the claims it added or changed.**

**Cold read.** You can't read the plan without this conversation, so hand it to someone who can: dispatch a read-only sub-agent and give it only the plan file's path. Ask it, for each step, whether it could carry the step out from the file and the code alone, and what it would have to guess. Don't pass it the objective, your concerns, or anything else from this conversation; that context is exactly what it must not have. With no sub-agent available, do the read yourself against Step 5's Self-contained rules and say so above the table.

Fix every step the reader couldn't act on, then give any claim the fix added its own row. Print every step in chat, not only the ones that changed:

| Step | Result |
|---|---|
| 1 | ✅ stands alone |
| 2 | ✏️ "<before>" → "<after>" |

Both tables are printed after their fixes are in the plan file, so what they show is what Step 8 presents.

## 8. Present

If Step 3 ended the objective and the user chose not to proceed, there is no plan. Skip `ExitPlanMode` and report what you found.

Precondition: Step 7's fixes are in the plan file, both tables are printed, and every claim in it has a row for its current wording. A claim without one → go back to Step 7.

Call `ExitPlanMode`. It takes no arguments and reads the plan file. Edit nothing else until it's approved.

## 9. Execute

Once the plan is approved, start implementing it without waiting to be asked. Work from the plan file, not from memory of this conversation: follow the steps in order and obey its `Constraints:` block — the branch, the stop-and-ask rule, the phase checkpoints, and the closing Verify run all live there.

If there is no plan (Step 8's no-plan branch), there is nothing to execute.
