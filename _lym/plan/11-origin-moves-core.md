# Card 11 — origin-moves-core

**Create:**
- `plan:internal/fabriccore#MoveTarget` -> `type MoveTarget struct`
- `plan:internal/fabriccore#MoveOrder` -> `func MoveOrder(targets []MoveTarget, pairs []string) (bolt, prime, pair []int)`
- `plan:internal/fabriccore#MoveFacts` -> `type MoveFacts struct`
- `plan:internal/fabriccore#MoveStep` -> `type MoveStep string`
- `plan:internal/fabriccore#MoveVerdict` -> `func MoveVerdict(facts MoveFacts) MoveStep`
- `plan:internal/fabriccore#RecordsMoveFacts` -> `type RecordsMoveFacts struct`
- `plan:internal/fabriccore#RecordsMoveState` -> `func RecordsMoveState(facts RecordsMoveFacts) fabriclayout.BranchOutcome`
- `plan:internal/fabriccore_test#TestMoveVerdict` -> `func TestMoveVerdict(t *testing.T)`
- `plan:internal/fabricengine_test#TestReconcileOrigin_PrimeLockHeld` -> `func TestReconcileOrigin_PrimeLockHeld(t *testing.T)`

**Edit:**
- `internal/fabricengine#originMover.moveAll`
- `internal/fabricengine#originMover.movePrime`
- `internal/fabricengine#originMover.move`
- `internal/fabricengine#reconcileRecordsBranch`
- `internal/fabricengine#tryPrimeLock`
- `internal/fabricengine#recordedPairs`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`

**Delete:**
- `internal/fabricengine#originMover.needsMove`

**Extends:**
- `internal/fabriccore/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#SetRunLocks`
- `plan:internal/fabricengine#RunLocks`
- `internal/fabricengine#ffWorktree`
- `internal/fabricengine#originMover.pairLive`
- `cmd/lym#uncontainedWriteAllowlist`

**Intent:**
The reconcile's moving half takes its order and its per-branch verdict from the core; the locks, the live-pair probe and the guards stay at execution in the engine (D4), and `originreconcilemoves.go` stops importing `internal/shedrun` (D13).

- `MoveOrder` is `moveAll`'s order: Bolt, then the prime's records before its code, then each pair's records before its code, in `pairs` order.
- `MoveFacts` holds what `move` decides on before acting: the judged state, the scope's run, the target's side and slug, whether the ended step owns the pair, whether another machine owns it, the busy lock name, and, once probed, whether the pair is live, whether a move record stands and whether the weft's anchor stops it.
  `MoveVerdict` answers, in today's order: no move for a branch that needs none (`needsMove`: fast_forward or needs_replay), one outside a run scope's own pair and Bolt, one the ended step owns, and a records branch needing a replay of an entry owned elsewhere;
  busy for a busy lock; probe live for a pair not the scope's own; live; anchor stopped for a standing move record or an anchor stop; fast-forward for a code branch; records move otherwise.
- `RecordsMoveState` is `reconcileRecordsBranch`'s verdict after its locks: busy naming the held lock; level; fast-forward; ahead; and after a replay level, ahead, dirty or diverged with the conflicts and local commits.
- The engine asks the live-pair seam (`pairLive`, once per pair) just before the move action, as an execution-time precondition, so a pair that went live after the judging snapshot is never moved; the core's verdict never replaces the probe (D4).
- `tryPrimeLock` takes its path from `RunLocks.PrimeRunLock` in place of `shedrun.PrimeRunLock`; `recordedPairs` reads through `RunRecordPaths.StatusFile`.
  Both keep their names and files, so `uncontainedWriteAllowlist`'s `tryPrimeLock` entry stays true, as do the readers map of `internal/fabricengine#TestBoltReplay_NamesNoBoardFile` naming `readOriginOwners` and `addLocalCreatedStamps`.
- `ffWorktree`'s dirty check runs at execution, against the live checkout, before the fast-forward.

Tests (the commit message lists them):
- `TestMoveVerdict` (core, tier0): `MoveOrder` over a hub's targets; `MoveVerdict` per rule above, in precedence order; `RecordsMoveState` per lock and replay result.
  It adds about 1 second to one tier0 run.
- `TestReconcileOrigin_Moves` keeps its rows that pin locks, the live probe and git effects:
  the hub fast-forward row, which now asserts the reconcile's exact printed mutations array (D4, D5) in place of `kindCount`;
  the pair weft clean and dirty row; the live-pair row, the tier1 test of D4's origin move refused when the pair went live after the snapshot;
  the held weft write and push lock rows; the push lock waited on up to the bound; the held write lock busy at once; the held Bolt push lock and prime weft write lock rows; the prime parent lock row through `PrimeParentLockFree`.
  It deletes the dirty prime weft and pair code way-forward row, the run-scope-judges-the-parent row and the move-record row, whose verdicts are `TestMoveVerdict` rows and whose lines are card 9's.
  Its prime lock row no longer holds `shedrun.PrimeRunLock`; `TestReconcileOrigin_PrimeLockHeld` holds it instead.
  Its rows that rest on run records, owners or the board's replay move in card 12.
- `TestReconcileOrigin_PrimeLockHeld` is a tier1 test in `internal/fabricengine_test`: it builds its hub through `internal/hubforge` first, sets `RunLocks` through `SetRunLocks` to a lock path in its own hub, restoring the installed seams on return, holds that lock, and does not call `t.Parallel`.
  A moving reconcile with the prime behind origin reports the prime's branches busy on the prime lock and moves nothing.

**ImpactSummary:** Private mover members change; originreconcilemoves.go drops its shedrun import; the allowlist entries naming its functions stay true; ReconcileOrigin's callers are untouched.
