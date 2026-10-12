# Card 2 — tmuxkit-run-directory

**Edit:**
- `internal/testkit/tmuxkit#Main`
- `internal/testkit/tmuxkit#afterRun` -> `func afterRun(w io.Writer, dir string, code int, grace time.Duration, sweep func(), remove func() error) int`
- `internal/testkit/tmuxkit#setEnv`
- `internal/testkit/tmuxkit#TestSetEnv`
- `internal/testkit/tmuxkit#TestListSockets_OwnDirectoryOnly`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`
- `internal/testkit/tmuxkit#TestCheckSockets`
- `internal/testkit/tmuxkit#TestSweep_KillsServersAndPaneTreesUnderItsDirectoryOnly`
- `internal/testkit/tmuxkit#TestAfterRun_LeftoverProcesses`
- `pattern/PATTERN-test-isolation.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/testkit/tmuxkit#kitBase`

**Create:**
- `plan:internal/testkit/tmuxkit#setTempEnv` -> `func setTempEnv(dir string)`
- `plan:internal/testkit/tmuxkit#TestAfterRun_ReportsFailedRemovalKeepingTheCode` -> `func TestAfterRun_ReportsFailedRemovalKeepingTheCode(t *testing.T)`

**Uses:**
- `plan:internal/lyxtemp#Dir`
- `plan:internal/lyxtemp#MkdirTemp`
- `plan:internal/lyxtemp#Remove`

**Intent:**
`tmuxkit.Main` uses the process run directory `lyxtemp.Dir()` as its private directory, the `TMPDIR`/`TMUX_TMPDIR` target, instead of its own `os.MkdirTemp(kitBase(), dirPrefix)`, and removes it after `m.Run` on every platform, Windows included (D4).
A `lyxtemp.Dir()` error is printed as `tmuxkit: ...` on stderr and returns 1 before any test runs.

Off Windows, `Main` keeps today's order: `checkSocketPath` on the run directory (on failure it removes the run directory through `lyxtemp.Remove` and returns 1), `setEnv`, `m.Run`, then `afterRun`.
On Windows, `Main` calls `setTempEnv`, leaves the tmux variables alone as today, runs `m.Run`, and passes the code through `afterRun` with a no-op sweep; the socket check finds no socket there, and the `/proc` scan stays Linux-only.
`setTempEnv` points `TMPDIR`, and on Windows `TMP` and `TEMP`, at the directory; `setEnv` calls it for its temp half instead of setting those variables itself, so both platforms set them through one function.

`afterRun` gains `remove`, called last in place of today's bare `os.RemoveAll(dir)`; `Main` passes `lyxtemp.Remove`.
A removal error, such as one a lingering child's cwd causes on Windows, is printed on `w` as `tmuxkit: remove run directory <dir>: <err>; the next start's sweep removes it` and leaves the returned code unchanged.
The next start's sweep removes that directory once its lock is free, since `lyxtemp.Remove` releases the lock before removing.

`kitBase` is deleted: its inherited-directory detection now lives in `lyxtemp`'s `resolveRootPath`, which places a re-executed helper's run directory beside its parent's.
Its two callers, `Main` and `TestSetEnv`, are on this card; `TestSetEnv` drops its `kitBase` assertions and keeps the ones on the variables `setEnv` writes.
Test sites that need a short directory of their own call `lyxtemp.MkdirTemp(dirPrefix)` instead of `os.MkdirTemp("", dirPrefix)`, the same depth as before since the old form already landed in `Main`'s directory: `TestListSockets_OwnDirectoryOnly`, `TestRemoveDeadSocket`, `TestCheckSockets`, `TestSweep_KillsServersAndPaneTreesUnderItsDirectoryOnly` and `TestAfterRun_LeftoverProcesses`, which also passes a `remove` that deletes its directory and keeps asserting the directory is gone after `afterRun`.

`TestAfterRun_ReportsFailedRemovalKeepingTheCode`, untagged in `tmuxkit_test.go`, calls `afterRun` over an empty directory with a no-op sweep and a `remove` returning an error, for codes 0 and 1, and asserts the returned code equals the input and `w` names the directory and the error; it covers the "removal a lingering child blocks is reported and leaves the exit code unchanged" criterion without a real child.

Docs in this commit: the package comment above `package tmuxkit` in `tmuxkit.go` states that `Main` runs in the process run directory, removes it after the run on every platform, reports a failed removal without changing the code, and that a killed binary's directory goes to the next start's sweep, replacing the `lyx`-prefixed temp directory sentence;
`pattern/PATTERN-test-isolation.md`'s "Private temp directory and leftovers" section says the same, names `internal/lyxtemp`, and replaces "Main creates its directory beside it" with the temp package's beside-the-parent rule.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/testkit/tmuxkit` row grows by the one new top-level test.

**ImpactSummary:** Every test binary under tmuxkit.Main now runs in and removes a lyxtemp run directory; afterRun's two callers are on this card.
