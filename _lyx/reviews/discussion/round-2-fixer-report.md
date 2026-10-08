# Fixer report — discussion round 2

## Fixed

- F1 (MEDIUM): confirmed against `internal/fabricengine/add.go`, which takes no lock the walk takes, and `commit.go`, which holds the weft write lock only inside `commitBothSides`.
  Decision 4: `lyx batten run`'s second call now always reconciles and commits its own new pair under the hub lock, whatever the stamp says, and names the remaining escape (a `lyx fabric add` whose fork read the stale prime and which becomes enumerable after the walk's last enumeration) with Loom-Preflight's retired-key refusal as its backstop.
  Decision 5: the prime is reconciled and committed before the first pair enumeration, and the holder re-enumerates before stamping until no new pair appears.
  Acceptance criteria: the batten-run criterion covers the fresh-stamp race; a new criterion covers a pair that becomes enumerable mid-walk.
- F2 (MEDIUM): confirmed that `Fabric.Commit` refuses at call time, after reconcile has written, and that `foreignMergeStatePresent` is unexported.
  Decision 7: the walk checks merge state (merge record and foreign git merge state, through one exported `fabricengine` query the plan adds) before reconciling and skips a mid-merge worktree unwritten; a commit that still refuses after the write restores the written files to their prior bytes and skips the worktree.
  New acceptance criterion for the restore.
- F3 (LOW): the cost risk in Open risks now states that every start verb and every `Preflight` row repeats the walk while a skipped worktree keeps the stamp stale.
  `lyx batten run`'s second call is now a single-pair call on every run, and its cost is stated in Decision 4.

Support log: a round-2 entry under Review rounds, three new rejected alternatives, and Q13 updated.

## Disputed

None.

## Deferred

None.
