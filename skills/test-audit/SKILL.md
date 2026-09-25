---
name: test-audit
description: "Invoke whenever writing, changing, reviewing, or sweeping tests in any language or framework. Authoring gate for new tests plus an audit workflow for low-value, implementation-coupled, or duplicative tests and the test-only production seams they demand."
---

# Test Audit

Three modes, one value bar.

- **Authoring mode** gates every new or changed test at write time.
- **Audit mode** runs focused sweeps for tests that re-assert source, duplicate
  stronger proof, couple behavior to implementation, or keep test-only
  production seams alive. Continue broad audits as separate coherent follow-up
  changes; optimize for confidence, not deletion count.
- **Campaign mode** prunes one whole subsystem's test surface (every test file a
  module, package, feature, or service owns). Before starting one, read
  [CAMPAIGN.md](CAMPAIGN.md).

## Project context first

This skill is technology-agnostic. Before judging or writing any test, learn
how *this* repository tests:

- Read the root and scoped agent instructions (`CLAUDE.md`, `AGENTS.md`,
  `CONTRIBUTING.md`, or equivalent) and any project knowledge base.
- Identify the test framework(s), the runner command for a single file or
  filter, the formatter and linter, and how CI selects and routes tests.
- Identify the test kinds in use (unit, integration, snapshot, UI/end-to-end,
  contract, property-based) and where each lives.
- Note project rules that override defaults here, such as "never run tests
  unless asked" or a required build tool. Project rules win.

Record these as the **project toolkit**; later sections refer to it.

## Authoring gate

Before adding any test, answer four questions; a missing answer means do not
add it yet:

1. What observable behavior, invariant, or independent contract does it protect?
2. What credible regression makes it fail?
3. Why does existing coverage not already catch that failure? Each contract has
   one primary test owner at the strongest boundary; another layer needs its
   own distinct risk, such as a transport or lifecycle failure the owner cannot
   reach. Prefer extending a parameterized/table-driven case or shared fixture
   over a near-duplicate test; consolidate duplicated setup in the same change.
4. Does it need a production seam (widened visibility, export, flag, wrapper,
   injection hook) that no production caller needs? If yes, move the test to
   the real boundary instead.

Then check the test against every [junk pattern](#junk-patterns); a match fails
the gate unless the [retention bar](#retention-bar) names the contract it
independently guards. A test that would break under behavior-preserving
refactoring is asserting implementation, not behavior; rewrite it at the
owning boundary before landing it.

Bug regression tests must fail on the pre-fix code for the intended reason and
pass after the owner-boundary repair. A regression test that never demonstrably
failed proves the mock, not the fix. One regression at the owner boundary
covers the bug; do not replay the same scenario at every layer it crosses.

## Junk patterns

The shared checklist for every mode: the authoring gate rejects a new test that
matches one, and audits hunt for existing tests that do.

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

## Value bar

Tests justify their maintenance cost by protecting behavior, a credible
regression, or an independently meaningful contract. In an audit, an existing
test that must change for behavior-preserving source reorganization is suspect,
not automatically deletable; the authoring gate still rejects new ones.

Before judging a candidate, read the complete test and production owner, its
entry point, callers, callees, sibling implementations, overlapping tests, CI
routing, and relevant history (`git log -p`, blame, linked tickets). When the
test claims dependency-backed behavior, inspect the dependency source or types
directly.

## Discovery

Keep discovery read-only and report evidence before editing. For broad scope,
run parallel discovery lanes when available, split along the repository's own
structure, for example:

- core libraries and shared packages;
- feature modules, plugins, or services;
- UI, apps, scripts, and tooling;
- a cross-cutting pattern sweep for the junk patterns.

Outside campaign mode, prefer a few high-confidence candidates over a large
speculative inventory. Hunt for the [junk patterns](#junk-patterns).

## Retention bar

Keep a test when it independently enforces a public API, SDK, protocol,
serialization format, config, migration, storage, security, platform, default
value, generated cross-language, package, release, or architecture contract.
Also keep:

- call ordering when order is observable behavior;
- regressions with a credible failure mode;
- source inspection when it is the cheapest independent guard: it fails when
  the contract changes (the user-facing key, byte, or path) and survives an
  identifier-only refactor;
- a retained test that fails on the baseline: treat it as a possible product
  bug, reproduce it, and repair the owner rather than deleting it.

Static or slow is not a deletion reason. A test that resembles implementation
may still be the independent contract; prove otherwise before removing it.

## Candidate evidence

Record every field below before editing. A missing field means the candidate is
not ready for deletion:

- exact test name and location;
- what failure it can actually detect;
- non-test callers of the covered production or support seam;
- stronger remaining owner-boundary proof, or why no proof is needed;
- relevant history and the reason the test or seam exists;
- production or test-support deletion unlocked;
- risk and the focused validation command from the project toolkit.

## Edit shape

Choose one coherent owner-boundary batch. Delete obsolete test-only exports,
visibility widenings, globals, wrappers, and dead production paths instead of
preserving aliases. Move retained regressions to their canonical owners.
Consolidate repeated package or dependency assertions into one generic
contract.

Prefer net-negative production LOC. Do not add replacement tests that restate
the same implementation, and do not convert uncertain candidates into cleanup
to increase deletion counts.

## Validation

Never edit source or tests while a test run or watcher is active in the same
checkout. Use the project toolkit's commands and respect its rules about when
building or testing is allowed.

1. Run the smallest owner and sibling tests (single file or filter).
2. For removed source greps or plan assertions, run the executable script,
   build step, or dry-run that owns the real contract.
3. Run the project's formatter and linter on changed files, then
   `git diff --check`.
4. Run whatever changed-files or affected-targets gate the repository uses in
   CI, if one exists locally.
5. Inspect `git diff --numstat`; report production/tooling separately from
   tests and test support.
6. After final audit edits, run a code review pass over the diff.

## Landing and continuation

Commit, push, open a PR, or land only when authorized, following the
repository's own branch and PR conventions. Land one coherent change at a
time; after landing, refresh from the default branch and rerun read-only
discovery for the next high-confidence batch.

## Handoff

Report:

- root cause and removed low-value categories;
- production owner simplifications;
- retained false positives and why they remain valuable;
- focused and full proof actually run;
- production versus test LOC;
- PR and merge state;
- named follow-ups.
