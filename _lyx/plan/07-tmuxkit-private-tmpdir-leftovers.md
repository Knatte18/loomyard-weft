# Card 7 — tmuxkit-private-tmpdir-leftovers

**Create:**
- `plan:internal/testkit/tmuxkit#afterRun` -> `func afterRun(w io.Writer, dir string, code int, grace time.Duration, sweep func()) int`
- `plan:internal/testkit/tmuxkit#leftoverGrace` -> `const leftoverGrace = 2 * time.Second`
- `plan:internal/testkit/tmuxkit#leftover` -> `type leftover struct`
- `plan:internal/testkit/tmuxkit#leftovers` -> `func leftovers(dir string) []leftover`
- `plan:internal/testkit/tmuxkit#Pids` -> `func Pids() []int`
- `plan:internal/testkit/tmuxkit#ProcArgv` -> `func ProcArgv(pid int) ([]string, bool)`
- `plan:internal/testkit/tmuxkit#ProcCwd` -> `func ProcCwd(pid int) (string, bool)`
- `plan:internal/testkit/tmuxkit#ProcExe` -> `func ProcExe(pid int) (string, bool)`
- `plan:internal/testkit/tmuxkit#IsWatchdog` -> `func IsWatchdog(argv []string) bool`

**Edit:**
- `internal/testkit/tmuxkit#Main`
- `internal/testkit/tmuxkit#setEnv`
- `internal/testkit/tmuxkit/tmuxkit_test.go`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `pattern/PATTERN-test-isolation.md`
- `pattern/PATTERN-testkit.md`
- `pattern/PATTERN-test-speed.md`

**Intent:** D8: a private temp directory per test binary and a leftover-process check.
`setEnv` also points `TMPDIR` (and `TMP` and `TEMP` on Windows) at the kit's directory, so every `t.TempDir`, `os.MkdirTemp("")`, lyxbin build and child temp file lands there; `TMUX_TMPDIR` stays the same directory.
`Main` no longer sweeps itself and returns `afterRun(os.Stderr, dir, code, leftoverGrace, func() { sweep(dir, os.Getuid()) })`, so card 10 can place its socket check ahead of the sweep inside `afterRun`.
`afterRun` runs `sweep`, waits `grace`, then on Linux collects `leftovers(dir)`: every pid whose cwd, exe or an argv element references `dir`; a pid whose argv `IsWatchdog` (an adjacent `reed`, `watchdog` pair) is killed through `proc.KillPID` without failing the package, since the watchdog idles out on `daemon-wakeups`' schedule; every other leftover is killed and the package fails with exit 1, naming pid and argv on `w`; then the directory is removed.
Off Linux the scan is skipped and the directory is removed.
The `/proc` probes live in a new file `internal/testkit/tmuxkit/proc.go` as exported helpers, `Pids`, `ProcArgv`, `ProcCwd`, `ProcExe` and `IsWatchdog`; they read files only and start nothing.
reedcli's own copies move onto them in card 8, and the copies in `loomcli`'s `smoke_helpers_test.go` and `orchcli`'s `smoke_live_test.go` stay, per the amendment "D8 moves only reedcli's probes; other copies stay".
Tests: `tmuxkit_test.go`'s `TestSetEnv` gains the `TMPDIR` assertion and a row that `t.TempDir()` lands under `os.Getenv("TMPDIR")`; `tmuxkit_integration_test.go` gains `TestAfterRun_LeftoverProcesses`, which starts under a temp directory a `sh -c 'sleep 60'` child with cwd in it and a second child whose argv carries `reed watchdog` after its script (`sh -c 'sleep 60' x reed watchdog`), calls `afterRun` with a short grace and a no-op sweep, and asserts exit 1 naming the first child's pid and argv and not the second's, both gone, and the directory removed; `IsWatchdog` rows sit in the same test.
`tmuxkit`'s budget row rises by one.
`tmuxkit`'s package comment records the private `TMPDIR`, the post-run `/proc` scan that kills a leftover and fails the package, the watchdog allowance and the exported probes, and widens its exemption bound to that scan and kill.
`PATTERN-test-isolation.md` gains a section "Private temp directory and leftovers"; `PATTERN-testkit.md`'s tmuxkit bullet widens its bound to the `/proc` scan and killing a leftover; `PATTERN-test-speed.md`'s "untagged tests spawn nothing" notes that the probes are barred from untagged files by the every-export rule although they spawn nothing, which costs nothing because an untagged test has no process to probe.

**ImpactSummary:** `Main` gains a post-run check every tagged package runs through, and `setEnv` moves every tagged package's temp files; no production code changes.

**Verify:** lyx gate test --tags tmux ./internal/testkit/tmuxkit
