# Card 11 — lyx gate test quits a Go test binary that keeps a core busy

**Create:**
- `internal/gatecli/watchdog.go`
- `internal/gatecli/watchdog_test.go`
- `internal/gatecli#enclosingGateEnv`
- `internal/gatecli#defaultSamplePeriod`
- `internal/gatecli#watchdog`
- `internal/gatecli#watchdog.run`
- `internal/gatecli#watchdogApplies`
- `internal/gatecli#TestWatchdog`
- `internal/gatecli#TestWatchdogApplies`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#runTiedChild` -> `func runTiedChild(cmd *exec.Cmd, started func(pid int)) error`
- `internal/gatecli#newCommand` -> `func newCommand(goBinary string, samplePeriod time.Duration) *cobra.Command`
- `internal/gatecli#Command`
- `internal/gatecli#testRequest`
- `internal/gatecli#standInRecord`
- `internal/gatecli#runStandIn`
- `internal/gatecli#runGateHelper`
- `internal/gatecli#startGate`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`
- `docs/overview.md`
- `cmd/lyx#unboundedWaitFiles`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/proc#TreeCPU`
- `internal/proc#IsAncestor`
- `internal/gateslot#WatchdogLine`
- `internal/gateslot#FindWatchdogQuit`
- `internal/gateslot#BusyQuitWayForward`

**Intent:**
D7 and D8: while go test runs, `lyx gate test` samples its child tree, and a `.test` binary that keeps at least 90 percent of one core busy for `busy_quit_sec` gets one SIGQUIT, so the Go runtime prints every goroutine's stack and go test reports the package failed long before its own `-timeout`.

`internal/gatecli/watchdog.go`:
- `watchdog` holds the window, the sample period, the sampler (`proc.TreeCPU` in production), the signaller (SIGQUIT to a pid through `os.FindProcess`), the clock (`now` and `after`, `time.Now` and `time.After` in production) and the writer the line goes to (the verb's stderr).
- `run(ctx, root)` logs the start through `internal/logger` with the window and period, then on each period samples the tree under `root`, the go process.
  It keeps per pid the previous CPU time and the busy time so far, for every process whose `Path` ends in `.test`; a pid's first sample is its baseline.
  A sample whose CPU delta is at least 90 percent of the elapsed clock time adds the elapsed time to that pid's busy time; any other sample resets it to zero.
  Once a pid's busy time reaches the window it sends one SIGQUIT, writes `gateslot.WatchdogLine` to the writer and logs the quit with binary, pid and window, and marks the pid so it never sends a second.
  The window therefore counts whole samples, and a window shorter than the period quits after the first busy sample.
  A sampler failure is logged and the next period samples again; `errors.ErrUnsupported`, which `proc.TreeCPU` returns off Linux (D8), is logged once as "no CPU watchdog applies on this platform" and ends `run`.
  It returns when `ctx` ends.
- `watchdogApplies` decides before go test starts, returning the reason it logs: no watchdog when the window is 0; none when the go test flags after `--` carry `-bench` or `-fuzz` (either dash count, with or without `=value`), since those keep a core busy by intent; none when `enclosing`, the inherited value of `enclosingGateEnv`, parses as a pid that `isAncestor` reports as an ancestor, since an outer `lyx gate test`'s watchdog already sees this run's test binaries.
  An empty, unparsable or non-ancestor value is ignored, such as one a tmux server a test started carried into its panes after leaving the outer run's tree.
  Any value naming a live ancestor skips the watchdog, a hand-set one naming the invoking shell or pid 1 included.
  An inherited slot does not turn the watchdog off: `verifytree.Verify` holds a slot and passes it to a verify command that calls `lyx gate test`, which is the outermost gate run there.

Wiring:
- `runTest` reads the window from the limits it already loads, `limits.BusyQuit`, or from the template config outside every hub.
  It calls `watchdogApplies` with the request's flags, `os.Getenv(enclosingGateEnv)` and `proc.IsAncestor`, and logs the reason.
  It sets `enclosingGateEnv` to its own pid in go test's environment alone, on every path, and passes `runTiedChild` a `started` callback that runs the watchdog in a goroutine over the child pid when it applies.
  After `runTiedChild` returns, it cancels the watchdog's context and waits for the goroutine to end.
- `runTiedChild` calls `started` with the child's pid after `proc.StartTied` returns and before `Wait`; a nil `started` is skipped.
- `newCommand` takes the sample period into `testRequest`, which gains a `samplePeriod` field; `Command` passes `defaultSamplePeriod`.
- `testRequest` also gains a `stderr io.Writer` field, which `newCommand`'s RunE fills from `cmd.ErrOrStderr()`; `runTest` hands `request.stderr` to the watchdog as its writer.
  `runTest` gains no parameter, so its signature is unchanged and an in-process `RunCLI` capture sees the line.

Bound: the watchdog signals only `.test` processes in the verb's own child tree, once each, and covers only verify commands that run through `lyx gate test`.
A CPU-bound package that is healthy but busy past the window is quit and fails, and the operator raises the key.
A run carrying `-bench` or `-fuzz`, a run under a live enclosing `lyx gate test`, a run whose inherited variable names a live ancestor, and a non-Go hub's verify are bounded only by go test's own `-timeout` or the verify timeout, as before.
`PATTERN-daemon-wakeups`: `run` wakes once per period, 5 seconds in production, and spawns nothing, so `internal/gatecli/watchdog.go` joins `cmd/lyx`'s `unboundedWaitFiles`.
`PATTERN-gate-slot`: the watchdog runs inside the held or inherited slot and changes no slot handling.
`PATTERN-spawn-observability`: the SIGQUIT and the sampler's start and failure are logged.

Tests in `watchdog_test.go`, untagged, with an injected sampler scripted per sample, a fake clock that advances one 5-second period per `after` call, a recording signaller and a buffer writer:
- `TestWatchdog` rows:
  - a `.test` process busy for the whole 10-second window is signalled exactly once, after its second busy sample, and the buffer holds exactly `gateslot.WatchdogLine` for it, from which `gateslot.FindWatchdogQuit` reads `10s`;
  - one sampled at 92 percent of one core for the whole window is signalled;
  - one that stays busy for further samples after its SIGQUIT gets no second;
  - busy, idle, busy samples restart the window, so no signal comes until two busy samples follow the idle one;
  - a `.test` binary whose path base is longer than 15 characters is signalled;
  - a busy process whose path does not end in `.test` is never signalled;
  - a sampler returning a plain error on its first call and then busy samples of a `.test` process for the whole window keeps sampling, signals that process exactly once and logs the sampler failure in the buffer of `logcapture.CaptureVerbose`;
  - a sampler returning `errors.ErrUnsupported` is called once, signals nothing, and `run` returns with one "no CPU watchdog applies" line in the buffer of `logcapture.CaptureVerbose`, since the line is logged at Info.
  `TestWatchdog` carries no `t.Parallel` and runs its rows in sequence, because the capture swaps the process-global logger; a top-level test without `t.Parallel` finishes before the package's parallel tests resume, so no other test logs into the capture.
- `TestWatchdogApplies` rows: a window of 0; `-bench=.`, `-test.bench`, `-fuzz=FuzzX` in the flags; a variable naming a pid the injected `isAncestor` calls an ancestor, all off; a variable naming a pid that is not an ancestor, an empty variable and an unparsable one, all on.
`internal/gatecli` has two top-level tests besides `TestMain` against a row of 3, so its budget row rises to 4.

`TestGateTest_Scenario`, integration-tagged:
- `startGate` strips `enclosingGateEnv` from the gate's environment, as it strips `gateslot.InheritEnv`, so the `lyx gate test` running this suite never makes a scenario's gate skip its own watchdog.
- `runGateHelper` passes a 1-second sample period to `newCommand`.
- `standInRecord` gains `Enclosing`, which `runStandIn` fills from `enclosingGateEnv`.
- The "a free slot" row asserts `Enclosing` is the gate process's pid.
- The "an inherited held slot" row asserts the nested gate's stderr holds the watchdog's start log.
  A second nested gate in the same slot, started with `enclosingGateEnv` naming this test process's pid, which is its ancestor, logs the enclosing-run reason and no start.
- A new row, Linux only, sets `busy_quit_sec: 1` in the shared `gate.yaml` and restores the file after.
  It writes a `spin` package into the nested module whose one test starts four goroutines spinning on a loop the compiler cannot remove, and spins itself.
  It runs the real `go test` there through the gate, and asserts all of: a non-zero exit; stdout holding `SIGQUIT: quit` and goroutine stacks; stderr holding the watchdog line, from which `FindWatchdogQuit` reads `1s`, the inner run's own window, which the outer suite's 120-second window cannot produce; and an end within 60 seconds, far below go test's 10-minute default timeout.

`gatecli`'s package doc gains a "CPU watchdog" section stating D7, D8 and the bounds above, and `## Exit codes` notes that a quit test binary's failure is go test's own code.
`pattern/PATTERN-daemon-wakeups.md`'s "What counts" list, which names each unbounded wait, gains the gate test's CPU watchdog, and the **gate** row of `docs/overview.md`'s module table gains that a Go test binary keeping a core busy past `busy_quit_sec` gets one SIGQUIT.

**ImpactSummary:** `runTiedChild` and `newCommand` are re-signed with their callers in `run.go`, `cli.go` and the integration helper; every `lyx gate test` now runs a sampler goroutine beside go test unless it is off, benchmarking, fuzzing or nested.

**Commit:** 11: lyx gate test quits a CPU-busy Go test binary with SIGQUIT
