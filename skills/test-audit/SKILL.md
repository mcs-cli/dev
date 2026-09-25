---
name: test-audit
description: >
  Gate new tests and audit existing ones for low value, implementation coupling,
  duplication, and test-only production seams, in any language or framework.
  Use when writing, changing, reviewing, or sweeping tests.
license: MIT
---

# Test Audit

Three modes, one value bar.

- **Authoring**: gate every new or changed test at write time.
- **Audit**: focused sweeps for tests that re-assert source, duplicate stronger
  proof, couple to implementation, or keep test-only production seams alive.
  Continue broad audits as separate coherent follow-up changes; optimize for
  confidence, not deletion count.
- **Campaign**: prune one whole subsystem's test surface. Read
  [references/campaign.md](references/campaign.md) before starting.

## Tool constraints

- Discovery is read-only: file reads, search, `git log -p`, `git blame`.
- Edit tests or source only after candidate evidence is recorded.
- Run only the test, build, format, and lint commands the project toolkit names.
- Never edit tests or source while a test run or watcher is active in the same
  checkout.
- Commit, push, or open a PR only when the user asks.

## Project toolkit

Before judging or writing any test, read the repo's agent instructions
(`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`) and knowledge base. Record the
test frameworks, single-file/filter runner command, formatter, linter, CI test
routing, test kinds and where each lives, and any rule that overrides this
skill. Project rules win.

## Authoring gate

Before adding any test, answer four questions; a missing answer means do not
add it yet:

1. What observable behavior, invariant, or independent contract does it protect?
2. What credible regression makes it fail?
3. Why does existing coverage not already catch that failure? Each contract has
   one primary test owner at the strongest boundary; another layer needs its
   own distinct risk, such as a transport or lifecycle failure the owner cannot
   reach. Prefer extending a table-driven case or shared fixture over a
   near-duplicate test; consolidate duplicated setup in the same change.
4. Does it need a production seam (widened visibility, export, flag, wrapper,
   injection hook) that no production caller needs? If yes, move the test to
   the real boundary instead.

Then check the test against every junk pattern below. Before failing a test on
a match, read the [retention bar](references/retention-and-evidence.md#retention-bar);
the match fails the gate unless the retention bar names the contract the test
independently guards. A test that would break under
behavior-preserving refactoring asserts implementation; rewrite it at the
owning boundary before landing it.

Bug regression tests must fail on the pre-fix code for the intended reason and
pass after the owner-boundary repair. A regression test that never
demonstrably failed proves the mock, not the fix. One regression at the owner
boundary covers the bug; keep it there instead of replaying it at every layer.

## Junk patterns

The authoring gate rejects a new test that matches one; audits hunt for
existing tests that do.

- assertion-free coverage probes (tests that only execute code to raise a
  coverage number);
- self-comparisons and identity copiers;
- copied fixtures, inventories, manifests, or export lists;
- exact source, import, or string greps;
- private predicate or call-shape tests duplicated at real boundaries;
- duplicate invocations of the same contract;
- per-implementation replays of a shared helper's behavior;
- tests whose only purpose is preserving test-only exports, globals, or wrappers;
- dead production code whose only callers are tests;
- expected values produced by the helper, formatter, or renderer under test;
- snapshots or golden files recorded from the code under test and never
  reviewed against an independent expectation;
- mocks that implement the asserted behavior, or one identical mock standing in
  for different APIs;
- fixtures that supply the result, ordering, or callback the owner should
  produce, or persistence asserted against a store the path never writes;
- capability tests that restate declared flags or configuration instead of
  exercising the behavior the flag promises;
- negative controls that pass for an unrelated reason, such as a denial from a
  different guard or a rejection the production path never reaches;
- names or fixtures that promise more than the input exercises, such as a
  "clears the cache" test asserting the cache was not cleared.

## Audit

1. **Value bar.** A test earns its maintenance cost by protecting behavior, a
   credible regression, or an independent contract. An existing test that must
   change for behavior-preserving reorganization is suspect, not automatically
   deletable.
2. **Read the full context.** For each candidate, read the complete test and
   production owner, entry point, callers, callees, sibling implementations,
   overlapping tests, CI routing, and history (`git log -p`, blame, linked
   tickets). When a test claims dependency-backed behavior, inspect the
   dependency source or types directly.
3. **Discovery.** Report evidence before editing. For broad scope, split
   parallel read-only lanes along the repo's structure: shared packages,
   features/services, UI/tooling, and a cross-cutting junk-pattern sweep.
   Outside campaign mode, prefer a few high-confidence candidates.
4. **Evidence.** Record every
   [candidate evidence](references/retention-and-evidence.md#candidate-evidence)
   field and check the retention bar before deleting anything.
5. **Edit shape.** Take one coherent owner-boundary batch. Delete obsolete
   test-only exports, visibility widenings, globals, wrappers, and dead
   production paths instead of preserving aliases. Move retained regressions
   to their canonical owners. Consolidate repeated package or dependency
   assertions into one generic contract.
6. **Validate**, using project-toolkit commands only:
   1. Smallest owner and sibling tests (single file or filter).
   2. For removed source greps or plan assertions, the script, build, or
      dry-run owning the real contract.
   3. Formatter and linter on changed files, then `git diff --check`.
   4. The repo's CI changed-files or affected-targets command, if the toolkit
      recorded one that runs locally.
   5. `git diff --numstat`, production/tooling separate from tests.
   6. A code review pass over the final diff.
7. **Land and continue.** Follow the repository's own branch and PR
   conventions and land one coherent change at a time. After landing, refresh
   from the default branch and rerun discovery for the next batch.

## Rules

- Static or slow is never a deletion reason.
- Prefer net-negative production LOC; skip replacement tests that restate the
  same implementation.
- Leave uncertain candidates in place and list them as follow-ups rather than
  deleting them to raise the count.
- A retained test failing on baseline is a possible product bug: reproduce it
  and repair the owner, keeping the test.

## Output

The task is done when the batch validates and this report is delivered:

- root cause and removed low-value categories;
- production owner simplifications;
- retained false positives and why they stay;
- focused and full proof actually run;
- production versus test LOC;
- PR and merge state;
- named follow-ups.

Example:

```text
Root cause: invoice tests asserted renderer output copied from the renderer itself
Removed: 6 source-grep tests, 2 mock-replay suites (billing/invoice)
Owner simplifications: dropped test-only InvoiceRenderer.reset()
Retained: invoice_currency_format (guards serialized wire format)
Proof: ./gradlew :billing:test --tests '*Invoice*' ✅, full :billing:test ✅
LOC: production -41, tests -312
PR: #123 open, CI green
Follow-ups: audit payments/refund lane
```
