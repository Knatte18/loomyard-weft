# Card 6 — verifyrun-tied-line

**Edit:**
- `internal/verifyrun#run`
- `internal/verifyrun#TestMain`

**Create:**
- `plan:internal/verifyrun#RunTied` -> `func RunTied(ctx context.Context, command, dir string, env []string, out io.Writer, started func(proc.TreeRecord)) (int, error)`
- `plan:internal/verifyrun#runTied` -> `func runTied(ctx context.Context, command, dir string, env []string, out io.Writer, delay time.Duration, started func(proc.TreeRecord)) (int, error)`
- `plan:internal/verifyrun#tieToParent` -> `func tieToParent(cmd *exec.Cmd)`
- `plan:internal/verifyrun#TestRunTied` -> `func TestRunTied(t *testing.T)`

**Uses:**
- `internal/proc#TreeRecord`
- `internal/proc#KillTree`
- `internal/proc#StartTime`

**Intent:** D6's tie of a card's line to the runner that runs it: on Linux the line's shell gets a parent-death SIGKILL, as `proc.StartTied` gives a tier, and the caller learns the line's process group to record it.

`runTied` is today's `run` body with one addition: when `started` is non-nil, it calls `tieToParent(cmd)` after `proc.ConfigureGroupTerminate` set the process group, runs the start and the wait on one goroutine locked to its OS thread (the parent-death signal follows the thread that started the child, as `internal/proc`'s tied start explains), and calls `started` with `proc.TreeRecord{PID: pid, PGID: pid, StartTime: <proc.StartTime of pid>}` once the shell runs and before the wait.
`run` delegates to `runTied` with a nil `started`, so `Run` behaves exactly as today; `RunTied` is `runTied` with `waitDelay`.
`tieToParent` sets `SysProcAttr.Pdeathsig = syscall.SIGKILL` on Linux, in a Linux-only file, and does nothing elsewhere, in a file for every other platform; on Windows the runner's own kill-on-close job ends the tree instead (card 13).
The package comment in `run.go` names `RunTied` beside `Run`; the import set is unchanged (standard library, `internal/logger`, `internal/proc`).

`TestRunTied`, tier1 and `!windows`, in a new test file whose build line is `//go:build tier1 && !windows`:
- `started` receives a record whose PGID equals its PID and whose start time is set, before the command exits, and the exit code is the command's own;
- `proc.KillTree` on that record ends a descendant the shell backgrounded (`sleep 30 & echo $! > pidfile; wait`), which the test then finds gone;
- a helper process runs `RunTied` on `sleep 30` and writes the record's PID to a file; the test SIGKILLs the helper and finds the shell gone, the parent-death signal's work.
`TestMain` gains the helper role for that row: when the helper variable names it, the binary runs the role and exits instead of running the tests.

**ImpactSummary:** `run` becomes a wrapper over a new body; `Run`'s behavior and every existing caller stay, and only the new `RunTied` sets the parent-death signal.
