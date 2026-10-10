# Card 12 — lyxbin-run

**Create:**
- `plan:internal/testkit/lyxbin#Run` -> `func Run(bin, dir string, timeout time.Duration, args ...string) (output string, exitCode int, err error)`
- `plan:internal/testkit/lyxbin#ErrTimeout` -> `var ErrTimeout = errors.New("lyxbin: the command outlived its timeout")`

**Edit:**
- `internal/loomcli/smoke_helpers_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_drivingsurface_test.go`
- `internal/loomcli/smoke_prebootstrap_integration_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_resumearming_test.go`
- `internal/reedcli/smoke_coldstart_test.go`
- `internal/reedcli/smoke_debuglog_test.go`
- `internal/reedcli/smoke_lifecycle_test.go`
- `internal/reedcli/smoke_selvage_keepalive_test.go`
- `internal/reedcli/smoke_staterecovery_test.go`
- `internal/reedcli/smoke_warmpath_test.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchcli/smoke_adopt_large_test.go`
- `internal/orchcli/smoke_live_test.go`
- `cmd/lyx/main_integration_test.go`
- `internal/testkit/lyxbin/lyxbin_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/proc#ConfigureGroupKill`
- `internal/testkit/lyxbin#Build`

**Prosa:**
- `pattern/PATTERN-testkit.md`

**Intent:** D10: tests run `lyx` through one runner.
`Run` runs `bin` with `args` in `dir` under `exec.CommandContext` bounded by `timeout`, in its own process group through `proc.ConfigureGroupKill`, inherits the process environment so the hermetic git variables reach the child, waits for it, and returns combined output, the exit code, and an error: nil on any exit, `ErrTimeout` wrapped with the args when the deadline killed it (exit code -1), or the start error; it calls no `testing.TB` method, so a concurrent caller can use it.
The per-package runners move onto it: loomcli's `runLoomCLINoFatal` becomes a one-line call to `Run` and `sharedLyxBinary` is deleted in favour of `lyxbin.Build`, which card 11 made once-per-binary; reedcli's `runReedCLINoFatal` wraps `Run` with its `t.Fatalf` on error and timeout; orchcli's `smokeRun` does the same; cmd/lyx's `exitSweepFixture.run` calls `Run` with a one-minute timeout.
`mustRunReed` stays, since it drives the in-process seam, not a binary.
Tests in `lyxbin_integration_test.go`: `TestRun_KillsTheGroupOnTimeout` runs the test binary itself in a sleeper role (an environment variable the file's `TestMain` honours, the way `gatecli`'s helper roles do) that starts a `sleep` child and sleeps, with a short timeout, and asserts `ErrTimeout`, exit code -1, and that both pids are gone; `TestRun_PassesTheEnvironment` sets a variable through `t.Setenv` and runs the test binary in a role that prints it, asserting the output.
`lyxbin`'s budget row rises by two.
`PATTERN-testkit.md`'s lyxbin bullet widens its exemption to "go build of ./cmd/lyx and running that binary under group kill".

**ImpactSummary:** Four package-local runners collapse onto one kit function; their callers are the listed test files and their behaviour under a normal exit is unchanged.

**Verify:** lyx gate test --tags tmux ./internal/loomcli ./internal/reedcli ./internal/orchcli ./cmd/lyx -- -run '^$'
lyx gate test --tags llm ./internal/orchcli ./internal/reedcli -- -run '^$'
lyx gate test --tags integration ./internal/testkit/lyxbin ./cmd/lyx -- -run 'TestRun_|TestExitSweep|TestRootHook'
