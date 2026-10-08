---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md Decision 3 and Decision 4
    summary: the stamp check runs in the root pre-run of every lyx command, but the operator placed it only in the start verbs that launch long-lived work
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md Decision 7, Open risks last item
    summary: per-worktree config writes are left uncommitted, which is the failure this task exists to remove; the operator requires the reconciled config to end committed
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: no criterion keeps a newly seeded pair from inheriting retired keys from the prime's committed weft config
---

### [BLOCKING] The check belongs in the start verbs, not in every command

**Location:** Decision 3 (root `PersistentPreRunE` of `cmd/lyx`) and Decision 4 (an opt-out annotation for commands that must stay instant).
**Issue:** The operator's design call (2026-10-08, after the board body was written) is that the stamp check does not run on every `lyx` command.
It sits only in the start verbs that launch long-lived work: `lyx orch start`, `lyx batten run`, `lyx loom start` and `lyx loom resume`, with the recipe's `Preflight` row as the safety net.
One reconcile there covers the prime and every pair of the hub, and `lyx orch start` reaching it first means the orch runs on the new config.
The record's every-command pre-run with an opt-out annotation is the opposite default; it also makes every unannotated command stop while a reconcile fails (Decision 9), the orch's own calls included.
**Fix:** Run the check from those four start verbs (and `Preflight`), through the one entry point of Decision 6.
Drop the root pre-run hook and the annotation's reconcile half; the stencil-seed annotation stays as it is.
Restate Decision 9's bound and the matching acceptance criteria for start verbs only.

### [BLOCKING] Reconciled config must end committed

**Location:** Decision 7 and the last Open risk ("Per-worktree config left uncommitted in the prime stays dirty").
**Issue:** On 2026-10-08 a manual reconcile left uncommitted `_lyx/config` changes that blocked a run twice: `Preflight`'s worktree-clean check refused (`binary-change-and-rules` itself), and Finalize's merge refused on a dirty worktree (`review-lens-fan`).
The operator's call is that the reconciled per-worktree config ends committed.
Leaving it for "that worktree's own records commit" reproduces both stops, so the task does not reach its goal of a hub that keeps working across a binary change.
**Fix:** Commit each worktree's reconciled config on its records side, through `fabricengine` as `lyx fabric commit` does, as part of the reconcile.
If committing into a running pair races its driver, the record must say how the race is avoided (for example under the pair's existing records lock), not leave the files dirty.
Add acceptance criteria: after the reconcile no worktree has uncommitted `_lyx/config` changes, and `Preflight`'s worktree-clean check and a Finalize merge pass.

### [BLOCKING] A seeded pair must not inherit retired keys

**Location:** Acceptance criteria.
**Issue:** The new pair `binary-change-and-rules` was seeded with `master_base` in `batcher.yaml` after the prime's working copy had been reconciled, because the prime's reconcile was uncommitted and seeding reads the committed weft config.
The operator's call is that a newly seeded pair never inherits retired keys from the prime.
No decision or criterion covers the seed path.
**Fix:** With F2's commit in place, add an acceptance criterion: a pair created after a binary change carries no retired key in any `_lyx/config` file, and its `Loom-Preflight` passes without a manual reconcile.
If seeding can still read a stale committed copy (for example a pair created before the prime's reconcile commit is pushed), state how the seed path reconciles or orders after it.
