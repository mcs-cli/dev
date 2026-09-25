# Test-pruning campaign

Campaign mode prunes one subsystem's whole test surface in one change set: a
module, package, feature, plugin, or service. The retention bar and candidate
evidence in [retention-and-evidence.md](retention-and-evidence.md) and the
junk patterns and validation in [SKILL.md](../SKILL.md) apply to every lane.
This file adds the order of work. Each step ends on its completion criterion;
do not start the next step early.

## 1. Baseline

Record the subsystem's test and test-support line counts and every test file's
pass/fail state at a pinned default-branch SHA. Keep baseline failures in their
own list: they are often real product bugs, not stale tests.

Done when every in-scope test file has a recorded baseline result.

## 2. Lanes and inventory

Split the surface into **lanes** along production owner boundaries, not file
names or prefixes. Typical lanes: configuration/accounts, commands or actions,
state and context, dispatch/routing, inbound handling, outbound handling,
persistence, transport/networking, UI, shared helpers, test harness, and
end-to-end or QA scenarios. Include the subsystem's cases at shared core
boundaries and its QA and live-proof harness tests.

Done when every test file and QA scenario the subsystem owns belongs to exactly
one lane.

## 3. Read-only ledger per lane

Give each lane to its own read-only agent. The agent reads every assigned test
in full, including parameter tables. It also reads the production owners and
their entry points, callers, history, and CI routing. Each test declaration
goes into a written **ledger** with one mark. A parameterized test is one
declaration unless its rows need different marks; then mark each row.

- `R`: retain, naming the contract and the bug it catches; a retained test that
  only moves to a better-named file stays `R` with the move noted;
- `F`: retain the contract but repair the assertion, such as a vacuous negative
  that passes when only one of several items is missing;
- `C`: consolidate, naming the owner that absorbs the assertion first: a sibling
  table case, a stronger boundary suite, or the shared owner in another package;
- `D`: delete, naming the proof that remains, or why no contract exists.

Judge a test by its assertions, not its name. A test named for clearing some
state may assert that the state was _not_ cleared.

Done when every declaration in the lane has a mark and an evidence line.

## 4. Layer plan per lane

Treat the per-test ledger as input, not as the edit list. A second read-only
pass, starting from the ledger, looks for the redundant **layer**: for example,
several suites replaying the same shared component through a mock, sitting
around stronger suites that exercise the real boundary. Name the **keeper**
suite for each contract. Prefer the real boundary with a fake dependency
(in-memory store, stub server, fake clock) over a mocked collaborator. Correct
any ledger errors this pass finds.

Done when each lane plan names its retired files, its keeper per contract, the
assertions to carry into keepers, and the test-only production seams unlocked.

## 5. Cutover

Edit lane by lane. Serialize changes to shared harnesses and support files
through one owner. With each lane, remove the test-only production seams it
unlocks: injection parameters, getters, reset hooks, widened visibility, and
indirection layers. Register moved suites in CI routing, build manifests, and
test inventories. Update any shrink-only size or coverage baselines. Put
durable test-ownership rules in the subsystem's agent instructions or
contributing docs, drawn from mistakes this campaign actually found.

Done when every lane plan is applied and each lane's keepers pass.

## 6. Preservation review

Before claiming completion, have independent reviewers compare deleted
coverage against the keepers, one reviewer per boundary group. They look for
contracts that lost their only proof. They also look for new assertions that
cannot fail, such as a rejection row the production code never reaches.

For each restored contract, make one deliberate **mutation** of the production
owner and confirm the keeper goes red. Then restore the source byte for byte.

Done when every reported gap is restored or rejected with source evidence, and
every restored contract has a caught mutation.

## 7. Product defects

A baseline failure that survives into a keeper is a bug report. Fix it at its
owner as a separate commit, and prove it through the real user flow, with a
**control** run that reverts the fix and shows the old behavior. Record
unrelated product discrepancies you find as follow-ups instead of fixing them
in the campaign.

Done when each repaired defect has a failing control and a passing candidate
on the same harness.

## 8. Reconcile and hand off

Campaigns outlive many default-branch commits. Merge the default branch rather
than rebasing a long, many-commit campaign, unless repository policy requires
otherwise. When the default branch modified a file the campaign deleted, keep
the deletion. Port the new contract into the keeper instead, and confirm every
new regression test the default branch added still has a home. Rerun the whole
subsystem suite and repeat live proof on the merged head.

Expect review tooling to see a truncated file list on a diff this large.
Record maintainer decisions about compatibility flags in the PR evidence
rather than editing gates.

Hand off with the [SKILL.md](../SKILL.md) report, plus:

- baseline and final test/support line counts, with production counted separately;
- lanes, retired layers, and keepers;
- preservation gaps found and their mutations;
- product defects with control and candidate proof.
