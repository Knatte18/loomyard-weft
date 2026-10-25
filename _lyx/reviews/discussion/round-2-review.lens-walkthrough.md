```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough list**

- D1 — `lyx reed down` of a beam session whose landing run is not running: `settledForSessionDown` reads settled, `endOwned` spares the `RoleDobby` record; purpose met.
- D2 — `lyx orch start` on a non-prime-owner machine: `lyx beam start` starts no prime dobby and warns; purpose met.
- D3 — run R taken over by machine B while machine A sleeps, A wakes: purpose **not met** (F1).
- D4 yield — run R at Run-Shed, child running: one check, `Yield`, nothing persisted; purpose met only if the one check is `check()` (F2).
- D4 spawn hold — run R held on a standing owner, no pause covering: purpose **not met** (F3).
- D4 in-pass bound — pass at Land running deploy, and a pass respawning a child via `lyx loom start`: bound's list incomplete (F4).
- D4 continue-pass refusal — hand `--continue-pass` with a full pass owed: refused, dobby runs a full pass; purpose met.
- D5 — a note upserted to `kind: task`: purpose **not met** (F5).
- D5 — `lyx fabric remove` of a halted run: `settleClaim` `claimClear` writes `draft`; purpose met.
- D5 — `lyx fabric remove` of an owned-elsewhere entry: `Local: true`, no board write; purpose met.
- D7/D4 — pass failing three times with one error text: one notice, retries go on; purpose met for an ordinary failure, but F1's case becomes an endless loop.
- D8 — Land failure: notice held until the Go driver hands the stop back; purpose met.
- D10 — landing finished by an orphaned pass while dobby is down, followed by `lyx cleanup --apply` before `lyx beam start`: purpose **not met** (F6).
- D12 — lift while a pass is in the land-lock poll: gets its lift pass on exit, as D12 says; purpose met.
- D13 — only halted runs: timer backs off; purpose met.

**Findings**

**F1 — BLOCKING, design, D3/D4 (owner takeover after a wake)**
A takeover becomes an endless failing pass with no owner pause, no lock release, no owner notice, and dobby never stops driving the run.
Walk: dobby sees the sleep and owes R a full pass → the full pass's `ownerPreflight` → `reconcileResume` → `resumeRefusal` returns `ownedElsewhereRefusal` and the pass exits non-zero (`internal/battencli/resumereconcile.go:80`, `internal/battencli/owner.go:211`) → the owed full pass stays owed, so dobby retries it on every probe tick → after three failures one failing-pass notice, "and the retries go on" (D4). InnerRun's `ownedElsewhere` (`internal/battenshed/innerrun_owner.go:105-118`) — `PauseOwned`, parent-lock release, the owner notice naming `lyx adopt` — never runs. D3's "It drives only runs this machine owns" has no mechanism by which dobby stops driving it.

**F2 — MEDIUM, design, D4 "one check of its child per Call"**
Today `wait()` runs coverage, `noticeStep` and `runArm` before its loop (`internal/battenshed/innerrun.go:669-678`); `foldOwnerTrigger` and `ownerStep` run only inside `check()` (`innerrun.go:700-710`), which the loop reaches after the first Sleep. If the pass's one check is the pre-loop round, the persisted `ownerDue`, pause-lift and retry triggers D4 carries across passes never fire. D4 does not say which round the one check is, or that it includes the owner fold and step.

**F3 — MEDIUM, design, D4 spawn hold returns `Yield`**
`holdSpawnWhileCovered` re-runs the owner check once per `noticeProbe` from inside its loop, after a Sleep (`internal/battenshed/innerrun_pause.go:47-69`). Yielding instead of sleeping returns before that retry, and before `wait`/`check` is reached (`innerrun.go:414-417`). Walk: run with a standing owner (`ownedBy` persisted, `PauseOwned` failed or pause lifted) → pass → spawn hold → immediate yield → repeat on every probe tick: the owner is never re-checked, so a run adopted back to this machine stays held forever.

**F4 — LOW, consistency, D4 "Bound: a pass stays in flight only through those waits"**
Two in-pass waits are missing from D4's list: InnerRun's respawn runs `lyx loom start --no-attach` synchronously with `cmd.Run` (`internal/battencli/wire.go:1205-1213`), and Land's deploy runs up to `deployTimeout` = 30 minutes (`internal/battenshed/land.go:33`, `:334`).

**F5 — MEDIUM, design, D5 "No code path but an explicit release leaves a task statusless"**
The board already counts "a note upserted to kind task" as making a task (`internal/boardengine/taskguard.go:19`); that upsert goes through `ApplyPatch` on the existing entry and keeps its nil status (`internal/boardengine/store.go:514-515`). D5's draft default covers "a write that creates a task" and `promote`, not this conversion, so dobby starts a statusless never-released task.

**F6 — LOW, design, D10 "a restart neither misses nor double counts one"**
Walk: a pass left by a dead dobby finishes Land; the operator runs `lyx cleanup --apply`, which removes the done run's directory, history included (`internal/battencli/cleanup.go:217`, `:276-277`); dobby restarts and never sees that Land Done, so the landing is uncounted.

**Tool calls**
1. Grep for `removeRunRecords` callers / shed persist functions — to learn whether batten run records survive a landing, which D10's count rests on.
2. Read `internal/battencli/cleanup.go` header and apply branch — to confirm when a done run's records are deleted.
