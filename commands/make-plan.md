---
description: Research, grill, and write a plan
---

# Make Plan

Turn an objective into a plan worth approving. Research first, settle the open decisions with the user, then write something short enough to read in one pass.

Arguments: $ARGUMENTS — the objective. Prefix with `quick` to skip the grilling round.

Think as deeply as the objective warrants — a one-file change needs far less than one crossing a module boundary.

## 1. Frame

**Call `EnterPlanMode` first**, unless already in plan mode. Nothing here touches project files — the mode is what guarantees it. The single exception is the plan file itself, written in Step 8.

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

`AskUserQuestion` is normally reserved for what you can't settle from sensible defaults. That bar doesn't apply here — the user ran this command to be asked. A decision you *could* default is still theirs to make.

## 5. Write the plan

**Concise by construction.** The per-section caps below are the budget; there's no global word count to game. A rename is a three-line plan, a twenty-file refactor is twenty one-line steps. A plan may run long because it has many steps — never because a section grew.

**Readable.** Write for someone deciding in a few minutes. Plain sentences, no throat-clearing, no paragraph whose real job is to look thorough. If a line could be cut without changing the decision, cut it.

**Shape:**

- **Why** — two sentences. The symptom, goal, or constraint. Don't restate the objective back at the user.
- **Steps** — numbered, one line each, naming the file and what changes in it. No paragraph under a step arguing the step is a good idea. Group them under phase headings **only** when something happens between the groups — a review, a deploy, a separate PR, a checkpoint where you'd stop and verify before continuing. Work that runs straight through is one ungrouped list, however long.
- **Verify** — how the user confirms it worked. Bullets, at most four, each a check plus its expected result. One line saying so if there's nothing to check manually.
- **Risks** — only when a step can fail in a way the user would want to hear about first. Omit the heading otherwise.

**No research narration.** What you learned while researching is not part of the plan. Where a `Context` heading is required — some plan-mode harnesses mandate one — the two-sentence Why goes under it and nothing else. Never let it grow into a summary of what you read; that belongs in the conversation or a memory file. This is the single biggest source of plan bloat.

Cut every sentence arguing *for* a decision already settled. The decision itself belongs in Step 6 as one line; the case for it belongs nowhere. A plan is what you'll do, not the argument that you should.

## 6. Carry the constraints

End every plan with this block verbatim, plus anything settled with the user — in the grilling round or earlier in the conversation. One line each, no rationale: the plan file outlives the conversation, so a decision that lives only in chat is lost.

```
Constraints:
- Comments explain why, never what. No comment that restates the line below it.
- Doc comments on public declarations only.
- Match the file's existing comment density.
```

Emit it even when the project or user instructions already say the same — where they do, those govern in full. Often they won't. These travel *with* the plan so they're re-read at implementation time: honor them while building, not just while writing.

## 7. Check the facts

Before presenting, re-verify every factual claim in the plan against the code, not against your memory of it: each file path exists, each named symbol exists where the step says it does, and each stated behavior is what the code actually does.

- A claim that doesn't hold gets fixed or dropped. Don't present a step you couldn't verify.
- If a wrong fact undercuts a decision the user made in Step 4, don't quietly re-decide it — put the correction to them with `AskUserQuestion` before presenting.
- Settled decisions are not facts to re-check. This pass verifies the plan, it doesn't reopen it.

## 8. Present

Write the plan where the harness expects it — plan mode names a plan file; write there — then call `ExitPlanMode`. It takes no arguments and no plan content: it reads what you wrote and signals that you're ready for approval. Don't edit anything else before it's approved.

If Step 3 ended the objective and the user chose not to proceed, there is no plan. Skip `ExitPlanMode` and report what you found.

## 9. Execute

Once the plan is approved, start implementing it without waiting to be asked. Follow the steps in order and honor the `Constraints:` block as you write code.

If the steps are grouped into phases, stop at the end of each phase, report what's done, and wait for the user before starting the next — that checkpoint is why the phase exists.

If there is no plan (Step 8's no-plan branch), there is nothing to execute.
