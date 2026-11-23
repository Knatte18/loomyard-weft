# Pin that a plan amendment during Webster keeps its done batches

Issue #185 reports a run that lost Webster's `state.json` after `lym loom decision add --amends-plan` and `lym loom goto --to Plan-Burler-Review`.
With the file gone, Plan-Burler-Fix's plan gate failed with `create-already-exists` on cards 1 to 4, whose Creates had already landed.
The incident's run cannot be traced: the issue names no slug or hub, and this hub's only Webster run in that window shows no deletion of the file.

This change adds `TestAmendsPlanDuringWebster_KeepsDoneBatches` in `internal/loomcli`, which drives the reported path end to end:

- Webster's run record holds four batches recorded done, each card's Create target exists in the tree, and a fifth card is pending.
- The real `decisionVerb` records a decision with `--amends-plan` and reads the plan's approval from disk.
- `shedengine.Goto` moves the halted run from Webster back to Plan-Burler-Review.
- The Plan-Burler-Fix plan gate, `loomshed.NewPlanGate`, then runs over the plan and the tree.

The test asserts that `state.json` keeps the bytes the batches wrote and that the gate passes with no `create-already-exists` on the done cards.
It passes on the current code, so no code change comes with it.

## Design decisions

- No step on the reported path moves `state.json`.
  The record moves in only three places: Plan-Write's and Parts-Plan-Write's plan-directory rotation, `lym webster run --fresh` and `lym webster reset --to start`.
  `decision add` writes the decision and amendment records alone.
  A goto writes the status file, and loom's goto hook resets the PR-Review record alone.
  Plan-Burler-Fix runs with overlay fix scope, so its gate reads the live anchor, where the record sits.
- With the run untraceable, the decision record's added entry "Untraceable incident driven in a test" has the path driven in a test,
  which lands when it shows no loss of the Webster run record.
- The entry is labelled `bug`, but the test passes on the merge base because the reported path has no defect to reach.
  The decision record's added entry "No test seam: the reported path loses no run record" lands it as a guard, not as a fix.

## Merge danger

**Door:** two-way
**Blast radius:** tests
