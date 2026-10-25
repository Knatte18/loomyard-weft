# Card 9 — drift-baseline-producer

**Create:**
- `plan:internal/driftengine#Deps` -> `type Deps struct`
- `plan:internal/driftengine#Row` -> `type Row struct`
- `plan:internal/driftengine#DriftBaseline` -> `type DriftBaseline struct`
- `plan:internal/driftengine#NewDriftBaseline` -> `func NewDriftBaseline(name string, row Row, deps Deps) *DriftBaseline`
- `plan:internal/driftengine#DriftBaseline.Call` -> `func (p *DriftBaseline) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/driftengine#awaitCaptureEnd` -> `func awaitCaptureEnd(ctx context.Context, p Paths, sleep func(context.Context, time.Duration) error) error`
- `plan:internal/driftengine#TestDriftBaseline` -> `func TestDriftBaseline(t *testing.T)`

**Edit:**
- `cmd/lyx#unboundedWaitFiles`

**Extends:**
- `internal/driftengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/driftengine#BaselineTable`
- `internal/driftengine#CheckMainCapture`
- `internal/driftengine#StartCapture`
- `internal/driftengine#RunCapture`
- `internal/driftengine#ErrBreakawayRefused`
- `internal/driftengine#CaptureHaltPrefix`
- `internal/driftengine#ProbeLock`
- `internal/driftengine#DirRel`
- `internal/shedadapters#NewMultiLLMProducerSourced`
- `internal/testkit/shedfake#SeatRunner`

**Intent:**
Add `Deps`, the told seams the three Drift producers share, `Row`, a row's portable config, and the `Drift-Baseline` producer (D3, D4).

`Deps` holds: the `Paths`; the worktree root; `Config func() (Config, error)`; `Status func() (shedengine.Status, error)`, the run's status; `AwaitRow bool`, true for the loom recipe, which has `Drift-Await`; `Head`, `BranchPoint` and `DirtyPaths` reads; `Commit func(relPaths []string) error`, which commits the told anchor-relative paths under `_lyx/drift` with the run's records; `Seats shedadapters.SeatRunner` and `Now`, which the registry entry fills from `Env.Seats` and `Env.Now`; `BaselineTable func(scopeNote string) (seatengine.Table, error)`; `StartCapture func() error` and `RunCapture func(ctx context.Context) (CaptureRecord, error)`, the detached spawn and the foreground run, built by the wiring over `StartCapture` and `RunCapture`; and `Sleep func(ctx context.Context, d time.Duration) error`, the wait's clock.
A nil seam a producer reads is a fail-loud wiring error naming the field.
`Row` holds `Subject` (`decision-record` or `board-entry`) and `WaitCapture` (darn's `capture: wait`).

`DriftBaseline.Call` commits through `Commit` on every way out: without `WaitCapture` only the scope record and `baseline.md` (`DirRel()` joined with `scope.json` and `baseline.md`), so no capture or capture-child log is committed while the detached capture still writes it, which `Drift-Await` commits in loom (D3, D4); with `WaitCapture`, and on the foreground fallback, all of `DirRel()` once the capture has ended:
with no scope record, or a scope record but no `baseline.md`, it runs one agent through `shedadapters.NewMultiLLMProducerSourced` over `Seats` and `BaselineTable`, the scope note saying the scope is already recorded when it is, so the agent writes `baseline.md` alone and its `lyx drift scope` refuses; an agent error or Stuck is returned as the seat producer maps it;
an agent that ends Done with no scope record is Stuck, `drift: the baseline agent recorded no scope; way forward: lyx loom resume, which spawns it again`;
scope `off` returns Done;
`awaited_at` set returns Done with no capture (D3);
a capture holding the capture lock is waited on through `Sleep` when `WaitCapture`, from one second doubling to thirty while the lock stays held (PATTERN-daemon-wakeups), and left running otherwise (Done);
a `passed` record is left alone (Done);
otherwise (no record, `failed`, or `running` with the lock free) it captures again:
`Config().Step("capture")` refusing is Stuck naming `lyx config drift` then `lyx loom resume`;
`CheckMainCapture` over `Status`, `Head`, `BranchPoint` and `DirtyPaths` refusing is Stuck with `CaptureHaltPrefix` and the refusal, whose ways forward name `lyx drift defer` with `lyx loom resume`, and on a dirty tree with HEAD on the branch point also restoring the listed paths to HEAD or stashing them, then `lyx loom resume`, which captures again (Shared Decision "a dirty capture halt names cleaning");
without `WaitCapture` it calls `StartCapture` and returns Done once the child's `running` record exists, so `Plan-Write` runs meanwhile, and on `ErrBreakawayRefused` falls back to the foreground run (D4); any other `StartCapture` error (a spawn that fails to start, a child that exits before its record) is returned as an error naming the child's log, a resumable stop whose resumed entry captures again, since the lock is free and no record passed;
with `WaitCapture`, or on that fallback, it calls `RunCapture` in the step's own process with no named job, so the command runs in the step's `StartTied` job tree and dies with the step, and returns Done on `passed` and Stuck on `failed`, the reason opening with `CaptureHaltPrefix` and naming the log, `lyx drift capture` followed by `lyx loom resume` to capture again, and `lyx drift defer` followed by `lyx loom resume` to go on with regression deferred.
The row carries no `on_stuck`, so each Stuck halts the run through the stop chain (D2), and none carries the decision halt prefix, so the arbiter may settle it with the way forward it names.

`awaitCaptureEnd`, in its own file `wait.go`, is that wait: it probes the capture lock, sleeps through the told `Sleep` from one second doubling to thirty while the lock stays held, and returns once it is free or the context ends; it has no bound of its own, since the capture enforces its timeout and releases the lock on exit (D5).
`Drift-Await` reuses it (card 10-drift-await-producer).
`cmd/lyx`'s `unboundedWaitFiles` gains `internal/driftengine/wait.go`, and `pattern/PATTERN-daemon-wakeups.md`'s list of unbounded waits gains the capture wait with its one-second floor and thirty-second ceiling.

`doc.go` gains the `Drift-Baseline` section.

`TestDriftBaseline` (tier0) is a table over fake `Deps` and a `shedfake.SeatRunner` whose run writes `baseline.md` and records a scope through `RecordScope`:
scope `off` returns Done and starts no capture; `partial` starts the detached capture and returns Done; with `WaitCapture` a passed foreground capture is Done and a failed one Stuck with the prefix naming `lyx drift capture` and `lyx drift defer`; re-entered with a scope record and no `baseline.md` the agent is spawned with the scope note; re-entered with both it spawns none; `awaited_at` set is Done with no capture; a lock held by the test is waited on under `WaitCapture` until released, then the record is read; without `WaitCapture` a held lock is Done with no capture started; a `running` record with a free lock captures again; a moved HEAD is Stuck naming `lyx drift defer`; a dirty tree on the branch point names restoring or stashing the paths, then `lyx loom resume`, and `lyx drift defer`, never removing or committing; `ErrBreakawayRefused` falls back to the foreground run, which on a failed capture is Stuck with the prefix naming `lyx drift capture` and `lyx drift defer`; another `StartCapture` error is returned as an error; an undeclared capture names `lyx config drift`; an agent ending with no scope is Stuck; `Commit` runs on every row's way out, told only the scope record and `baseline.md` without `WaitCapture` and all of `DirRel()` with `WaitCapture` and on the foreground fallback.

**ImpactSummary:** One new file joins the daemon-wakeups scan list; no existing code path changes.
