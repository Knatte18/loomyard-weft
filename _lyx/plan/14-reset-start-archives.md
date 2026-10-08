# Card 14 — reset-start-archives

**Edit:**
- `internal/websterengine#PlanReset`
- `internal/websterengine#ResetPlan`
- `internal/websterengine/doc.go`
- `internal/websterengine/reset_test.go`
- `internal/websterengine/reset_integration_test.go`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli/reset_test.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine#ArchiveRunAfterReset`
- `internal/websterengine#UncommittedPaths`
- `internal/websterengine#RemoveRecoveryStrands`

**Intent:** `reset --to start` becomes the one start-over verb: it ends by archiving the run record, and archives without moving when the start cannot be moved to (D3), listing what it left uncommitted (D4's envelope half).

The pair reset to start: after the branch reset and the state save succeed, `resetCmd` calls `ArchiveRunAfterReset` (card 12) under the state-mutation lock it already holds, carries the guard's drop warnings in a `warnings` envelope key, and reports `moved: true`.
When the guard refuses after a move, the move and the state save stand and the record stays unarchived; re-running the reset converges, since a reset to the commit HEAD is already on moves nothing.
A moving pair reset puts HEAD on the start and discards the run's tracked changes, so there the guard can still refuse only on an uncleared contract file, a differing plan path, or a suspect path left as an untracked file.

The archive-only path: when the start cannot be moved to, `PlanReset` returns a plan marked archive-only (`ResetPlan` gains `ArchiveOnly bool` and `Reason string`) instead of refusing, for exactly this closed set: no batch recorded a start, a recorded start is missing from the repository, the recorded starts share no single oldest commit and no common ancestor, or the resolved start is not an ancestor of HEAD.
It keeps the refusals that come before the target resolves (run lock held, no `state.json`, merge in progress, foreign branch) and drops the missing-commit, no-common-ancestor, ancestor and dirty-tree refusals for `start` only, which guard a branch move it does not make; `pre-fix` and the card-11 targets keep all of them.
The verb removes the run's live recovery strands first exactly as a moving reset to start does (refusing with the `lyx reed remove` way forward when one cannot be removed), runs no git that mutates (listing the uncommitted paths, below, is a read), archives through `ArchiveRunAfterReset`, and reports `moved: false` with `reason`.
Because card 12's guard judges pending paths against HEAD and reads no recorded commit (the shared decision `reset-guard-judges-against-head`), none of the four causes reaches a guard refusal; the guard refuses here only on an uncleared contract file, a differing plan path or a suspect path differing from HEAD, each with a step that clears it, then the reset again.
Bound: the archive-only path moves the run record into archive names, never deletes it, and moves no commit or file of the tree; it lets through a branch rewritten under the run and any uncommitted changes, which the next run starts over, and drops a pending finding only once its paths match HEAD or no verb can check them, with a warning per finding.

The uncommitted list: every reset to start carries `UncommittedPaths` (card 13), read after the move, in an `uncommitted` envelope key, so the operator sees what the next run starts over: in a pair the run's own tracked changes are discarded and untracked files stay; the archive-only path keeps everything.

`resetCmd`'s long help: the refusal list says `start` archives without moving where it named the no-recorded-target, missing-commit and not-an-ancestor refusals; "It clears the persisted pre-fix head and changes no other webster state" becomes a statement that `--to start` also archives the run record behind the pending-findings guard, and names the `moved`, `reason`, `uncommitted` and `warnings` keys.
The file header of `internal/webstercli/reset.go`, the file that holds `resetCmd`, says the verb clears the pre-fix head; it gains that `--to start` ends by archiving the run record, or archives alone when the start cannot be moved to.

Tests, one assertion per behavior at the surface that owns it:
- `TestPlanReset_Resolution` in `reset_test.go` (card 11's table over the package's `fakeGit`) gains a row per archive-only cause a fake `Git` can produce (no recorded start, a missing start, a non-ancestor start), each asserting the returned plan's `ArchiveOnly` and `Reason`; the pre-run refusals stay pinned by `TestPlanReset_HeldRunLockIsTransientBusy`, `TestPlanReset_NoStateNamesRun` and the integration steps.
- `TestPlanReset` in `reset_integration_test.go` drives `PlanReset` over a real repository for the one cause a fake cannot produce, recorded starts with no common ancestor (`octopusMergeBase` runs git), returning the archive-only plan.
- `TestResetCmd` in `webstercli/reset_test.go` extends its existing "start resets head and own dirt and keeps untracked" step to assert the record is archived (a following state load finds none) and the `moved: true` and `uncommitted` keys, and adds an archive-only step under a rewritten branch asserting `moved: false`, `reason` and the record archived.
  Its "no recorded target" and "target not an ancestor" refusal rows are kept for `pre-fix` only, since `start` no longer refuses there.

`refusal-spec.md`, by row:
- `reset: no recorded target, start`: deleted, the archive-only path replaces it.
- `reset: target commit missing`: reworded to apply to `pre-fix` and the card-11 targets, stating that `start` archives without moving instead (`moved: false`).
- `reset: target not an ancestor of HEAD`: its "for `start`, run `lyx webster run --fresh`" clause is deleted and the row states that `start` archives without moving instead; the `pre-fix` and card-11 clause stays.
- `reset: dirty tracked path the run did not write`: reworded so it does not fire on `start`'s archive-only path.
- `fresh: recorded starts diverge`: its "the reset resolving the commit every recorded start descends from" clause becomes "the reset moving to the commit every recorded start descends from, or archiving without moving when none exists"; card 17 then rewrites its step pair.
The package doc's reset section states the archive, the archive-only set and the per-mode trees.

**ImpactSummary:** Changes what reset --to start leaves behind (an archived run record, behind the pending-findings guard) and turns four of its refusals into an archive-only success; pre-fix and the card-11 targets behave as before.
