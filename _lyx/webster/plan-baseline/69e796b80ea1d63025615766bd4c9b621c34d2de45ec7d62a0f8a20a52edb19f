# Card 33 — dobby-passes

**Edit:**
- `plan:internal/dobbyengine#Deps`
- `plan:internal/dobbyengine#daemon`
- `plan:internal/dobbyengine#daemon.tick`
- `plan:internal/dobbyengine#RunState`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/dobbyengine#RunView` -> `type RunView struct`
- `plan:internal/dobbyengine#PassExit` -> `type PassExit struct`
- `plan:internal/dobbyengine#passKind` -> `type passKind int`
- `plan:internal/dobbyengine#planPass` -> `func planPass(run RunView, rs RunState, trigger passTrigger, covered bool) passKind`
- `plan:internal/dobbyengine#passTrigger` -> `type passTrigger struct`
- `plan:internal/dobbyengine#daemon.drivePasses` -> `func (d *daemon) drivePasses(ctx context.Context, now time.Time) error`
- `plan:internal/dobbyengine#daemon.passExited` -> `func (d *daemon) passExited(slug string, full bool, code int, errText string, at time.Time)`
- `plan:internal/dobbyengine#TestPlanPass` -> `func TestPlanPass(t *testing.T)`
- `plan:internal/dobbyengine#TestDrivePasses_ExitHandling` -> `func TestDrivePasses_ExitHandling(t *testing.T)`

**Uses:**
- `plan:internal/dobbyengine#PassExitCovered`
- `plan:internal/dobbyengine#PassExitContinueRefused`
- `plan:internal/dobbyengine#PassExitOwnedElsewhere`

**Intent:**
Schedule batten passes for every run whose parent worktree is the dobby's own (D3, D4, D7, D12).
`Deps` gains the run listing, from the prime's local batten status files, never the board, with each run's parent worktree as its pair's origin record names it (`hubgeom.ParentOf`, the prime for none), so a run whose task Land already pruned is still driven through Worktree-Teardown; and per run a `RunView`: batten's state and current producer, its status file's modification time, the child status file's modification time, whether the slug's cancel marker exists, whether the run's driver lock is held, whether its origin-owed marker stands, and whether a pause entry covers the hub or the slug; plus `SpawnPass(slug string, continuePass, noReopen bool) (<-chan PassExit, error)`, which starts `<lyx> batten run <slug>` with the hidden flags in its own process group with the prime's anchor (`Geometry`'s prime anchor) as its working directory and logged at Info, and reports its exit code and last error text as a `PassExit`, the struct carrying both;
the pass resolves its location through batten's own pre-run, which needs a worktree anchor, as the window run's `OpenWindowIn(..., c.location.AnchorPath(), ...)` and orchcli's spawn (`cmd.Dir = c.location.AnchorPath()`) do today, while the hub root, which is no git repository, stays the dobby's own working directory (card 27's `SpawnDetached`, D1); dobby registers no record of its own for the pass, since the pass registers itself in `internal/runproc` as the run's batten through `registerSeededRun`.
A dobby start owes every run it drives a full pass, and each sleep the loop records owes every run one; an owed full pass is kept in the state record and cleared when a full pass of that run exits 0 with no origin-owed marker standing, so a failed one is retried as a full pass; a run with its origin-owed marker standing is owed a full pass, run with `--no-reopen`.
`planPass` decides, for one run and trigger, none, a continue pass or a full pass: none unless batten's state is `running`, none while the cancel marker exists or a pass is in flight (a held driver lock reads as in flight, so a pass left by a dead dobby is waited for and never doubled), only continue passes and the full pass at the lift while a pause covers the hub or the slug, and for a run the state names owned elsewhere only the passes the next card names; a continue pass needs the run at Run-Shed with no full pass owed, and every other pass is full.
The triggers are a write of the child's status file, a write of batten's own status file while no pass of the slug is in flight and with a modification time later than the slug's last pass exit (so a pass's own writes never trigger the next, even when their event arrives after the pass exits), a probe tick once per `notice_probe_s`, the dobby's start, and the lift of a pause entry that covered the run, which runs a pass at once (or as soon as the pass in flight exits); a run held at a stop by the Go driver's pause hold goes on by itself at the lift and its in-flight pass gets no second one.
`passExited` reads the exit: 0 clears the run's failing-pass marker and its owed full pass as above; `PassExitContinueRefused` owes a full pass; `PassExitCovered` is a covered pass, counted in no failure; `PassExitOwnedElsewhere` hands the run to the next card's owned-elsewhere handling and is never a failing pass; any other code counts a failure by its error text and the pass is retried on the next probe tick, and three failures in a row with the same text post one notice per run and text to the nearest live orch (`<slug>: batten pass keeps failing: <text>; way forward: fix the cause the text names, then run lyx batten run <slug>; lyx batten status <slug> shows the run`), the texts noticed kept in the state record and cleared when a pass of the run exits 0.
A halted or paused batten run gets no pass; the orch or the operator resumes it with a hand pass.
Bound: per run at most one pass per `notice_probe_s` plus one per write of the child's status file, and at most one in flight; a run counts as in flight for the loop's backoff while `planPass` could give it a pass on the next probe tick.
The doc states the scheduling rules and the bounds.
Tests (tier0, fakes, parallel): `TestPlanPass` rows: running at Run-Shed with nothing owed gives a continue pass; owed gives full; origin-owed gives full with no reopen; halted, cancel-marked and in-flight runs give none; under a covering pause a continue pass on a probe tick, none for an owed full pass, and a full pass at the lift; a batten status write no later than the last pass exit gives none; a held driver lock with no pass this dobby spawned reads as in flight and gives none; a lift while a pass is in flight gives the run its pass once that pass exits. `TestDrivePasses_ExitHandling` rows over a fake `SpawnPass`: a dobby start owes every listed run a full pass and a re-exec owes none; a recorded sleep owes every run a full pass; a refused continue pass owes a full pass and counts no failure; a full pass that exits 0 while the origin-owed marker stands stays owed; a failed full pass is retried as a full pass on the next probe tick; a covered exit counts no failure and sends no notice; three failures with one text send one notice and a fourth none; an exit 0 clears the count and the noticed text; a run whose board entry is gone is passed from its status file alone; a run dropped from the listing is no longer passed; a dobby whose runs are all halted passes the idle flag to `nextWait`.

**ImpactSummary:** The engine's scheduling core and its seams; no caller outside the package yet.
