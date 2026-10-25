```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I ran each changed function or branch on one concrete input. Two of the cases I ran show a defect (overview `acting` check, done run with a pass in flight), and one lock-probe concern is a likely defect I could not confirm.

**Per function or branch, case run, verdict**
- `released`, `heldByEntry` and `readyToStart` (`starts.go`): a task with a nil status, an empty status, a draft status and a pending dependency. This matches the old predicate. Correct.
- `BuildView` runs: a done run, a run owned elsewhere, and a run with a held driver lock. A done run is dropped. A run owned elsewhere stays listed but takes no slot. Correct, except for the in-flight done run in F2.
- `BuildView` slots with `max_runs` 0: every task that nothing else holds gets `AtMaxRuns`. `startReady` returns early and starts nothing. Correct.
- `BuildView` slots with 1 free slot and 3 tasks (one held by a dependency, two free): the held task takes no slot, the first free task takes it, and the second is marked `AtMaxRuns`. This matches the `startReady` loop, which breaks at the top. Correct.
- `BuildView` refusal: the entry changed since the refusal, so the fingerprint differs, and the view shows no refusal. Backoff still running shows `NextTry`. Backoff expired shows the text with no `NextTry`, and `Ready()` is true. This matches `startReady`. Correct.
- `refusedStart` and `updateStart`: a first refusal writes `Refusal` with the backoff. A fingerprint change or a claim resets the whole `StartState`. The delete-when-empty case now includes `Refusal`. Correct. The key is optional (`omitempty`).
- `slotsTaken` and `runsInFlight`: same count as before the refactor. Correct.
- `logLine`, `startFirstPass` and `spawn`: a nil `Log` does nothing. A failed spawn logs nothing, because the log call sits after the error return. Each start or pass writes one line. `startFirstPass` does not go through `spawn`, so nothing is logged twice. Correct.
- `passKind.String`: all three kinds are named, and anything else prints `none`. Correct.
- `DobbyTasks`: a task with an empty status and a pending dependency now gets its parent and owner read. A task with a non-empty status gets neither. `PendingDependencies` lists only dependencies still on the board and not done, in `DependsOn` order. A pruned dependency counts as done, as before. Correct. A `ParentOf` or `Effective` error for a blocked task now fails the whole read, which is the decided bound.
- `DobbyRuns`, `dobbyParentRuns`, `DobbyLandings`, `lockHeld` and `modTimeOf`: the logic is unchanged from `beamcli`. Correct.
- `hubDobbyDeps`: the `Log` seam writes to the `out` writer passed from `runDobby`. Correct.
- `DobbyViewReader.Read`: a missing state record reads as the zero `State`. A missing dobby directory reads as not alive. Any other failure returns a wrapped error. Correct.
- `DobbyLines`: an empty view, a dead dobby, and a run with a zero `LastPassExit` (renders `no pass yet`). Correct.
- `dobbyTaskWait`: with several reasons they join with `; `. A task with an expired backoff and `AtMaxRuns` shows only `max_runs`. Correct.
- `overviewSource.frame`: `dobby` nil draws nothing. Otherwise it appends a blank line and then the dobby lines. Correct.
- `dobbyStatus`: the prime comes first, then each beam. One unreadable dobby does not stop the others. Correct. `runVerb` puts the result into `ErrFields` on error, so the per-entry errors reach the envelope.
- `statusCmd`: `acting` is unused and the verb acts on the prime, which matches the decision. Correct.

**Findings**

- **F1, MEDIUM, design:** `internal/battencli/overview.go:526-543`. The overview adds a dobby section for any acting worktree whose directory exists.
  - **Evidence:** `acting := location.WorktreeName` is passed straight to `NewDobbyViewReader`. `hubgeom.DobbyGeometry` (`internal/hubgeom/dobbygeom.go:23-50`) fails only when the directory is missing. It does not check that the worktree is the prime or a beam.
  - **Effect:** `lyx batten overview` run in a task pair draws `dobby <slug>: not running; way forward: lyx beam start`. That contradicts the acceptance criterion that a task pair's frame carries no dobby lines. It is reachable only by running the verb by hand, since `lyx orch start` ensures the pane only in the prime and in beams. No test covers this.
  - **Fix:** only attach `source.dobby` when `acting` is the prime or in the list from `hubgeom.DobbyStart(prime).Beams()`, and add a test row for a task pair.

- **F2, LOW, design:** `internal/dobbyengine/view.go:418`. `BuildView` drops every done run, but the scheduler keeps a done run in `d.runs` while a pass of it is in flight (`internal/dobbyengine/passes.go:149-152`).
  - **Effect:** while a done run's last pass (such as Land) still holds its driver lock, `slotsTaken` in the scheduler counts it and the view does not. A task can show as ready while `startReady` holds it back, which the view should never do.
  - **Fix:** keep a done run in the slot count when `DriverLockHeld` is true. It need not be listed in `view.Runs`.

- **F3, MEDIUM (uncertain), design:** `internal/battencli/dobbyreads.go:303-306` and `:175-186`. Every overview poll takes a momentary write lock on the dobby lock (`dobbyengine.Live`) and on each driven run's driver lock (`lockHeld`).
  - **Evidence:** the old `beamcli` code did this once per dobby tick. The pane now repeats it each poll in the prime and in every beam, on top of the dobby's own probes. I did not confirm whether the `lyx beam dobby` startup and the `lyx batten run` driver lock retry on a non-blocking refusal.
  - **Effect:** a pane probe can briefly hold a lock a starting dobby or pass tries to take, so the dobby or pass could take it as another instance already running. This also contradicts the doc comment's "without the records' locks" and "reads files only".
  - **Fix:** probe liveness from the lock's file state, or the pid in the state record, without taking the lock. If a probe is kept, confirm the acquirers retry.

**Tool calls**
1. `sed` on `internal/dobbyengine/starts.go`, `grep d.runs`, and `grep DobbyGeometry`: my context lacked the `startReady` body and `DobbyGeometry`, which I needed to compare the view with the scheduler and to check whether `DobbyGeometry` rejects a task pair.
2. `grep runVerb`: my context lacked `runVerb`, which I needed to see whether the result survives an error.
3. `sed` on `internal/dobbyengine/passes.go` and `grep func Live`: my context lacked the done-run filtering in `drivePasses` and the body of `Live`.
