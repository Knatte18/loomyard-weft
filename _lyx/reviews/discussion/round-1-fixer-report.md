# Fixer report — discussion round 1

Every finding was checked against the code and fixed in `_lyx/discussion/decision-record.md`, with ledger and rejected-alternative updates in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` passes.

## Fixed

- **F1 (BLOCKING): a mid-merge prime breaks the no-inherited-retired-key promise.**
  Confirmed: `Fabric.Commit` refuses with `ErrMergeInProgress`/`ErrForeignMergeState` before taking any lock (`internal/fabricengine/commit.go`), and the prime is in the walk.
  Decision 4 now says that when the first walk leaves the stamp stale because it skipped a worktree, the prime included, `lyx batten run` calls the entry point a second time after creating or adopting the pair and before its driver starts, so the new pair is reconciled and committed.
  A failure of that second walk stops the verb as Decision 9 states.
  Decision 7 points at Decision 4 for the skipped-prime case, and the acceptance criterion covers the skipped prime.
  The alternative, stopping `lyx batten run` on a mid-merge prime, is in the support log's Rejected alternatives.
- **F2 (MEDIUM): Preflight's stop is missing from Decision 9's bound.**
  Decision 9's bound now says every run's `Preflight` goes `Stuck` too, a fresh run or a `goto` re-entry alike, even when the failing file is in an unrelated pair.
  Decision 10 names `Preflight`'s own way forward: fix the named file (or `lyx config <module>`), then `lyx loom resume` in the task worktree.
  The acceptance criterion covers the unrelated-pair case.
  The restating Open risk was removed (see F5), since Decision 9 now carries the full blast radius.
- **F3 (MEDIUM): Decision 14's role membership contradicts itself.**
  Confirmed: reviewer, judge, describer and reflection stencils all write their own artifact files.
  The set is now every spawned role other than the orch, through its opening stencil, the same per-role set `PATTERN-parent-directive` pins (`roleOpeningStencils` in `contracts/stencils/stencils.go`); a fork inherits it through its parent's opening stencil.
  The Decision states that the rule constrains the tool, not the write surface, so it never widens a role's write limits.
  Scope, the Decision's title, the PATTERN-entry wording, the acceptance criterion, the Auto-mode line and ledger Q8 all use that definition.
- **F4 (LOW): the hub-lock wait has no bounded duration to sit above.**
  Confirmed: `Fabric.Commit` and the merge verbs take the combined write lock with the blocking `lock.AcquireWriteLock` (`internal/lock/lock.go`).
  Decision 5 now states that the holder's per-worktree waits are unbounded, as `Fabric.Commit` is today, so the hub-lock wait is the only bound a waiter has.
  The plan picks that wait above a walk with no contended pair lock, and a waiter that times out behind a healthy holder stops, leaves the stamp untouched, and finds the fresh stamp when re-run.
  Decision 10 states `Preflight`'s hub-lock timeout: `Stuck`, with the way forward to run `lyx loom resume` once the other reconcile ends, under its own refusal-spec row and reaching test.
  The Constraints line and the acceptance criterion now cover both `Preflight` refusals.
- **F5 (NIT): relocation candidates.**
  Decision 14's rationale is cut to the one fact the Decision doesn't state (the rule lives in a chat skill spawned roles never load).
  The rejected PATTERN-directive and `scribe:conversation` routes stay only in the support log.
  The two Open risks that restated Decision 9's bound and Decision 7's mid-merge retry were dropped.

## Disputed

None.

## Deferred

None.
