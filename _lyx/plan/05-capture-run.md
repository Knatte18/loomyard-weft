# Card 5 — capture-run

**Create:**
- `plan:internal/runproc#RoleCapture` -> `const RoleCapture = "capture"`
- `plan:internal/driftengine#CaptureHaltPrefix` -> `const CaptureHaltPrefix = "drift capture halt: "`
- `plan:internal/driftengine#CaptureRequest` -> `type CaptureRequest struct`
- `plan:internal/driftengine#RunCapture` -> `func RunCapture(ctx context.Context, req CaptureRequest) (CaptureRecord, error)`
- `plan:internal/driftengine#SideResult` -> `type SideResult struct`
- `plan:internal/driftengine#RunSideCapture` -> `func RunSideCapture(ctx context.Context, req CaptureRequest) (SideResult, error)`
- `plan:internal/driftengine#CheckMainCapture` -> `func CheckMainCapture(status shedengine.Status, head, branchPoint string, dirty []string) error`
- `plan:internal/driftengine#SpawnRequest` -> `type SpawnRequest struct`
- `plan:internal/driftengine#ErrBreakawayRefused` -> `var ErrBreakawayRefused = errors.New("drift: the capture start was refused breakaway")`
- `plan:internal/driftengine#StartCapture` -> `func StartCapture(req SpawnRequest) (int, error)`
- `plan:internal/driftengine#CaptureJobName` -> `func CaptureJobName(worktreeRoot string) string`
- `plan:internal/driftengine#ProbeLock` -> `func ProbeLock(path string) (bool, error)`
- `plan:internal/driftengine#Active` -> `func Active(p Paths) (bool, error)`
- `plan:internal/driftengine#TestRunCapture` -> `func TestRunCapture(t *testing.T)`
- `plan:internal/driftengine#TestCheckMainCapture` -> `func TestCheckMainCapture(t *testing.T)`
- `plan:internal/driftengine#TestStartCapture` -> `func TestStartCapture(t *testing.T)`

**Edit:**
- `internal/runproc/doc.go`
- `internal/gateslot/doc.go`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `internal/proc#StartTied`
- `internal/proc#Tied.Kill`
- `internal/proc#DetachBreakaway`
- `internal/proc#IsBreakawayRefused`
- `internal/runproc#ChildRecord`
- `internal/runproc#Register`
- `internal/gateslot#Pool.Acquire`
- `internal/gateslot#HolderEnv`
- `internal/gatecli#runTiedChild`
- `plan:internal/driftengine#WriteCapture`
- `plan:internal/driftengine#ReadCapture`
- `plan:internal/driftengine#Config.Step`

**Intent:**
Build the capture run (D4): the body the detached `lyx drift capture-child` runs, the side capture, the spawn of the detached child, and the lock probe every reader uses.

`CaptureRequest` carries the told `Paths`, the worktree root, the declared `capture` `CommandStep`, the gate-slot pool and wait dir, the scope's skip list, HEAD's commit, a `Check func() error` precondition run under the lock, a `Timeout` (the step's `timeout_min`, which a test shortens), a `Now` clock and a `LockRetry` bound (production a few seconds, a test shortens it).

`RunCapture`, the main capture:
it takes `CaptureLock` with a non-blocking take retried until `LockRetry`, so a reader's probe never makes it refuse, and refuses past it with `drift: a capture already holds <lock>; way forward: wait for it to end (lyx drift status shows it), then lyx loom resume`;
runs `Check` and returns its refusal untouched, writing no record;
writes a fresh `running` record with HEAD and its own pid;
moves a non-empty `BaselineDir` to the first free numbered backup beside it (`baseline.1`, `baseline.2`, …), so no baseline is deleted or overwritten;
writes the skip file (one test per line, empty when nothing is skipped);
acquires one slot as a non-tier holder (`gateslot.Holder{Worktree, Site: "drift capture"}`) with no deadline, writing a `gateslot.WriteWait` record while it waits, and records `slot_acquired_at`;
renders `{baseline_dir}` and `{skip_file}` with `verifyrun.QuoteArg`, and runs the command through `verifyrun.ShellFor`'s shell in `dir` (absolute as is, else joined onto the worktree root), with the process environment, the step's `env` and `gateslot.HolderEnv`'s slot variables, its stdout and stderr the open log file `LogsDir/capture-<UTC stamp>.log` itself (D4: no copying pipe);
starts it through `proc.StartTied` and logs the spawn through `internal/logger` (PATTERN-spawn-observability), records `command_started_at`, and at `Timeout` counted on the monotonic clock from the command's start kills the whole tree with `Tied.Kill`, as `gatecli`'s `runTiedChild` ends a timed-out run;
once the command is gone it records `passed` for exit 0 with at least one file under `BaselineDir`, else `failed` with the reason (`exit <code>`, `exit 0 but wrote no file to <dir>`, or `timed out after <n>m`), then releases the slot and the lock.
The command is an unregistered child of the capture process, so ending it touches no `runproc` record.
`RunCapture` holds no job object of its own, since it also runs in a row's step process (card 9-drift-baseline-producer), where releasing a kill-on-close job would end the step; the detached child holds the capture's named job around it (card 7-drift-verbs-cli).

`RunSideCapture` (D6) takes no capture lock, writes no capture record, needs no clean tree or branch point: it renders `{baseline_dir}` as the next free numbered directory under `SideDir`, holds one slot (`Site: "drift side capture"`) and runs as above, returning the directory, the log path and the exit code.

`CheckMainCapture` is the main capture's precondition (D4), called by the capture child under the lock and by `Drift-Baseline` before it starts one:
the status's current row is `Drift-Baseline`, or `Drift-Await` while the status is `blocked`;
HEAD equals the branch point;
the tracked tree is clean.
Each refusal opens `drift capture refused:` and names its way forward: the row refusal names running `lyx drift capture` only at a `Drift-Await` halt or a `Drift-Baseline` capture halt, which `lyx loom status` shows;
a moved HEAD names `lyx drift defer` followed by `lyx loom resume`;
a dirty tree names restoring the listed paths to HEAD or stashing them (never removing them, which leaves a tracked deletion, and never committing them, since a commit moves HEAD off the branch point), then the next step the told status's row gives: at `Drift-Baseline` `lyx loom resume`, which captures again, and at a `Drift-Await` halt `lyx drift capture` again, then `lyx loom resume`, since the resumed `Drift-Await` only reads the records; both also name `lyx drift defer` followed by `lyx loom resume` (Shared Decision "a dirty capture halt names cleaning").
`CaptureHaltPrefix` opens every `Drift-Baseline` Stuck reason on a failed or refused capture (D3); `CheckMainCapture` also admits a `blocked` status at `Drift-Baseline` whose error opens with it.

`StartCapture` first refuses while `ProbeLock(CaptureLock)` reads held, with `RunCapture`'s own refusal text, so a caller never mistakes another capture's lock for its child's.
It spawns `<executable> drift capture-child` detached in the worktree root, with `proc.DetachBreakaway`, the child's stdout and stderr the open file `LogsDir/capture-child-<UTC stamp>.log`, so a refusal the child meets before its capture log exists (the lock take, `Check`) is kept; a start Windows refuses for breakaway is retried with `proc.Detach` when `SpawnRequest.FallbackDetach` is set (the `lyx drift capture` caller, living as long as its shell's job) and returns `ErrBreakawayRefused` otherwise (the row, which then runs the capture in the foreground, card 9-drift-baseline-producer).
It logs the spawn, registers `runproc.ChildRecord(pid, runproc.RoleCapture, argv)` under the run's slug in the told run-process directory (a failed registration is logged at Warn, as `boardengine`'s sync spawn does), and returns the pid only once the capture record reads `running` with the child's pid, which the child writes after `Check` passed under the lock, polling from a told start interval.
So the caller moves the run on (the row's Done, a parent's `lyx loom resume`) only after the child's precondition held, and the child's `Check` never reads a status the start itself changed.
A child that exits before that record is an error naming the child's log, and so is a start the bound outlasts; each ends with its caller's way forward, told by `FallbackDetach`.
For the `lyx drift capture` caller (`FallbackDetach` set) it is: read the named child log, then, once `lyx drift status` shows the capture lock free, run `lyx drift capture` again, or `lyx drift defer` followed by `lyx loom resume`.
For the row (`FallbackDetach` unset), whose error stop admits no `lyx drift defer`, it is: read the named child log, then `lyx loom resume`, whose entry captures again.
`SpawnRequest` carries the executable, worktree root, `Paths`, run-process directory and owner, `FallbackDetach`, a record-wait bound, a `spawn` seam over the start (nil starts the real command) so a test replaces the child, and a `breakawayRefused func(error) bool` seam (nil is `proc.IsBreakawayRefused`) so a Linux test reaches both breakaway branches.

`ProbeLock` takes the lock without blocking and releases it at once, answering held; a missing directory reads as not held and creates nothing.
`Active` is `ProbeLock(CaptureLock) || ProbeLock(CompareMarker)` (the compare marker's writer is card 6-compare-run).
`CaptureJobName` is `lyx-drift-capture-` and a hex hash of the worktree root, as `shedrun.LoopJobName` names a loop's job.

`runproc`'s `doc.go` role list gains `RoleCapture`; `gateslot`'s `doc.go` non-tier holder list gains the drift capture and side capture.
`driftengine`'s `doc.go` gains the capture section.

Tests:
`TestCheckMainCapture` (tier0) is a table over each admitted row and state, the capture-halt `blocked` at `Drift-Baseline`, a `blocked` status at another row, `running` at `Drift-Await`, a moved HEAD, and a dirty tree at `Drift-Baseline` and at a `Drift-Await` halt, asserting each refusal's way forward, the dirty rows each their row's wording (restoring or stashing, then `lyx loom resume` at `Drift-Baseline` and `lyx drift capture` then `lyx loom resume` at `Drift-Await`).
`TestRunCapture` (tier1, `!windows`, fake `sh` commands, a temp pool) has rows: a command writing a file passes and the record holds the times, its pid and log; under a skip list the command sees `{skip_file}` naming a file that holds one skipped test per line, and with none an empty file; exit 0 with no file fails; a non-zero exit fails; a command sleeping past a shortened `Timeout` kills its child tree and records `failed`, after which the lock and slot are free; an existing non-empty baseline is moved to `baseline.1` and kept; a second capture refuses while the lock is held past `LockRetry`; a lock held briefly by a probe inside `LockRetry` does not make it refuse; with the pool's only slot held, the record stays `running` with no `slot_acquired_at` until the slot is released; `Check`'s refusal writes no record; a side capture writes `side/1`, leaves `capture.json` and `baseline/` untouched.
`TestStartCapture` (tier0) drives the `spawn` and `breakawayRefused` seams: the argv is `drift capture-child`, the record is registered with role `capture`; the call does not return while the fake child holds the lock with no record or an older record of another pid, and returns once the child writes `running` with its pid; a child that exits without that record is an error naming the child's log and, with and without `FallbackDetach`, its caller's way forward; a fake child that holds the lock and never writes its record past a shortened record-wait bound is the same error; a lock held before the start refuses with nothing spawned; a breakaway refusal returns `ErrBreakawayRefused` or, with `FallbackDetach`, retries detached; `ProbeLock` on a missing directory reads free and creates nothing.

**ImpactSummary:** One new `runproc` role constant and two doc lists; no existing caller changes.
