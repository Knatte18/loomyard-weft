```yaml
lens: code-correctness
outcome: reported
```

Task: review the code-correctness lens of this round's diff.

**Per-function list (case run → verdict)**

- `WrapRowGate` — an entry name absent from the spec returns the way-forward error; with the entry present, the other entries get wait-only wrappers → correct.
- `rowWrap.spawning`, boundary failure, no live record, held=0 → the fixer spawns, is awaited, the gate re-runs and its verdict carries the reply → correct.
- `rowWrap.spawning` bound: held = attempts+1 → returns the failure, no spawn → correct.
- `rowWrap.spawning` with a failing forked result for which `SkipForked` is true → no spawn → correct.
- `rowWrap.spawning` with a pass, pending or terminal result → returned as is → correct.
- `rowWrap.spawning` with an inner error → returned, no spawn → correct.
- `fixerEntry` Final path: `Final` is `waitOnly(inner)`, so it never spawns, and the forked choice follows `rf.Forked` → correct.
- `waitOnly` as the first entry (Darn `clean`) with a live record → waits, runs, marks terminal → see finding 1.
- `waitThenRun` with the inner closure erroring → the record is still marked terminal → finding 2.
- `spawnAndJudgeAgain`, `SpawnOrAttach` fails → the failure stands with a "no fixer ran" line → acceptable, finding 5.
- `wait`, `Await` error, stop ok → record marked dead/died → correct.
- `wait`, `Await` error, stop fails → record stays live, joined error returned → correct.
- `expired`, live record not spawned by this wrap → true (restart); live record spawned by this wrap → false → correct as an approximation.
- `rowStore.Live`, `Save`, `count` and `markTerminal` (missing guid errors) → correct.
- `SpawnOrAttach` — a role outside `Roles` is refused; a live record is attached; the Start, FindRun, Register and Save failures are stopped or abandoned → correct.
- `Await` — checks run in the order reply, turn ended, timeout, strand gone, as D1 says; a reply read failure leaves an empty reply with only a warning → correct, with a minor edge.
- `rowVisitDir`, empty `History` → repeats 0, key `row-0-0`; a missing status file → the way-forward error → correct.
- `gateFixerSeam`, empty pair slug → the spec comes back unchanged → correct.
- `gateFixerSeam`, `FixerTimeout() &lt;= 0` → the branch is unreachable in practice, finding 6.
- `endGateFixers` and `fixerEndingGuard` — an empty slug ends nothing, a refusal from the inner guard ends nothing, and an end error refuses the move → correct.
  - A relaunch with an empty target never runs the guard (`goto.go` runs it only for a non-empty `Target`), so it ends no fixers.
  - A targeted relaunch ends fixers before `Admit`, which can still refuse.
  - Neither case leaves a live fixer.
- `LensReviewOnly` — findings are joined from the start by "; ", so the regex heads line up; only lens-review → true, mixed → false, no blocking finding → false → correct.
- `handsOff` — verb, empty slug, fixer role, no gate flag or empty `fixer` all return false; an empty `Role` hands off → correct.
- `settleLiveFixer` — a live record with the kit on and the wait ok → marked terminal, gate runs; the kit off or the wait failing → stop, then mark dead only after the stop succeeds → correct.
- `settleLiveFixer`, record unreadable → finding 4.
- `fixAndRerun` — an abandoned phase stops the fixer; a wait error stops it, and a failed stop exits 1 with the record left live → correct.
- `fixAndRerun`, record marked terminal before the re-run → finding 3.
- `fixAndRerun`, re-run output: `runRequest` recreates stdout and stderr (`os.Create`, `detachrun.go:333-334`), so the re-run replaces the first run's output and the reply is appended → correct.
- `joinable` fixing skip applies only while no exit is written, and `rewriteTreeIdentity` runs before phase `starting` → correct.
- `verifyFixerStore` and `FixVerifySpawnOrAttach` — a live record is attached before the report preconditions are checked, and Save appends → correct.
- `Registrar` — an empty owner returns nil; an absent strand or a zero pane pid is an error → correct.

**Findings**

1. **MEDIUM, design.** `waitOnly` marks the fixer record terminal after any wrapped entry's inner closure, whether it passed or failed.
   - Location: `internal/gatefixer/rowgate.go:117-131`, via `waitThenRun` at `:199-214`.
   - Evidence: on Darn-Write, a restart with a live fixer makes `clean` wait first, pass and mark the record terminal.
   - Consequences:
     - The `verify` closure then runs with no live record, so a failing verdict carries no fixer reply.
     - A second restart while `verify` runs finds only a terminal record, so `expired` is false and nothing finalizes.
   - D2 says the record is marked terminal by the first entry that fails over the finished tree, or by the fixer entry once its closure returns. Marking it on a passing earlier entry contradicts that, and the restart-during-re-run guarantee (D1, acceptance "restart … judges the re-run") is lost.

2. **LOW, design.** `waitThenRun` marks the record terminal even when the inner closure returned an error (`internal/gatefixer/rowgate.go:204-205`). D1 says terminal comes once a result is held, so a driver error during the re-run leaves a terminal record and the next visit never re-waits or re-judges.

3. **LOW, consistency.** The card runner marks the fixer record terminal before the re-run starts (`internal/gatecli/detachfixer.go:206-212`). D1 says terminal comes once a result judged over the finished tree is held. The fixer has already ended, so no harm follows from the early mark, but the code departs from the decision.

4. **LOW, design.** When `store.Live()` fails to read the record, `settleLiveFixer` logs a warning and runs the gate anyway (`internal/gatecli/detachfixer.go:123-127`). D4 says a run attaches or stops a fixer it cannot attach to before the gate, never running over its half-done edits. An unreadable record skips that rule.

5. **LOW, design.** `spawnAndJudgeAgain` turns every `SpawnOrAttach` error into a findings line and returns `nil` (`internal/gatefixer/rowgate.go:183-187`).
   - The transient mark `SpawnOrAttach` puts on a start failure is discarded.
   - No record is saved, so the attempts+1 bound does not count these tries.
   - Each later boundary evaluation retries the spawn, which is bounded only by the writer's attempts.

6. **NIT, design.** `gateFixerSeam` refuses `FixerTimeout() &lt;= 0` with a "run lym config reconcile" way forward (`internal/loomcli/gatefixer.go:209-211`). `decodeConfig` already refuses `fixer_timeout_min &lt; 1` at load (`internal/gateslot/config.go:245`), so the branch looks unreachable in production.

7. **LOW, design, verify.** `fixAndRerun` fires on any non-zero, non-abandoned first exit (`internal/gatecli/detachfixer.go:112`). A call refusal, a usage error or a slot-shortage exit therefore also spawns a fixer, not only a red verdict. I did not read the exit-code map, so check it against D4's "red verdict".

**Tool calls**
1. `grep` of `internal/loomshed/gates.go` for how plan findings are joined: the manifest held only the `LensReviewOnly` diff, not the format.
2. `grep` of `internal/shuttleengine/wait.go` for `MayHold`, `Final`, `Attempts` and `Expired`: the manifest lacked the shuttle semantics.
3. `grep` of `internal/gatecli/detachrun.go` for the stdout/stderr open modes: the manifest lacked `runRequest`.
4. `grep` of `internal/shedengine/goto.go` for the guard call sites: the manifest held only the doc-comment diff.
5. `Read` of `internal/loomshed/gates.go` lines 20-50: to confirm the "; " join and `Finding.Error`.
6. `Read` of `internal/loomshed/gates.go` lines 225-265: to confirm the plan-gate findings text begins with the first finding.
