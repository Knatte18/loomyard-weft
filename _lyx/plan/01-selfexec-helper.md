# Card 1 — selfexec-helper

**Create:**
- `internal/selfexec/doc.go`
- `internal/selfexec#Watcher`
- `internal/selfexec#identity`
- `internal/selfexec#New`
- `internal/selfexec#newWatcher`
- `internal/selfexec#Watcher.Check`
- `internal/selfexec#statIdentity`
- `internal/selfexec#execSelf`
- `internal/selfexec#errUnderTest`
- `internal/selfexec#TestWatcher_Check`
- `internal/selfexec#TestLeafInvariant_AllowlistOnly`
- `internal/selfexec#TestMain`

**Edit:**
- `pattern/PATTERN-leaf-packages.md`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`

**Intent:** Add `internal/selfexec`, the shared self-re-exec helper of D8, as a leaf over the standard library and `internal/logger` only.

`New` resolves `os.Executable` once and records that path and its file identity (device, inode, size, modification time) through `statIdentity`; a failed resolve or stat logs at Debug and returns a watcher whose `Check` never execs.
`Check` re-stats the recorded path, never re-resolving it, since on Linux a replaced binary's `/proc/self/exe` names a deleted file (the model is `exeWatch` in `internal/reedcli/watchdog.go`).
A failed stat counts as unchanged, logs at Debug and resets the streak.
An identity that differs from the start identity and equals the identity seen at the previous check (two consecutive checks) logs at Info that the process replaces itself, naming the path, then calls the exec seam with the recorded path, `os.Args` and `os.Environ()`.
A failed exec logs at Warn and records that identity as failed, so no later check execs for it again; a later identity that differs from it is tried again after its own two consecutive checks.
The exec seam `newWatcher` takes is `execSelf` in production: it returns `errUnderTest` without exec'ing when `testing.Testing()` reports a test binary, and otherwise calls `syscall.Exec`, so nothing execs under `go test`.
The Windows build (a build-tagged file beside the Unix one) has no exec: its `statIdentity` returns an error, so every `Check` reads as unchanged and the process keeps running.
The exec runs only the recorded path with the process's own argv and environment, so it can start nothing the process was not already running; that is the helper's whole bound.
Per `PATTERN-spawn-observability`, the Info line before the exec is the spawn's log; per `PATTERN-log-levels` nothing a poll repeats logs above Debug.

`doc.go` carries the package doc: what the watcher records, the two-check rule, the failure levels, the test and Windows guards, and that its callers are long-lived pane processes that call `Check` from their own poll, adding no wake of their own (`PATTERN-daemon-wakeups`).

`TestWatcher_Check` is a table over `newWatcher` with a fake stat sequence and a recording exec seam, rows:
an unchanged identity never execs;
a changed identity seen once never execs;
a changed identity seen on two consecutive checks execs once with the recorded path, `os.Args` and `os.Environ()`;
a changed identity that flips back before the second check never execs;
a failed stat between two changed checks never execs, since it resets the streak;
a failed exec keeps the process running, logs one Warn line (captured with `internal/testkit/logcapture`) and is not called again on further checks of that identity;
a later, different identity after a failed one execs again after two checks.
One more row calls `execSelf` itself with a path that does not exist and asserts `errors.Is(err, errUnderTest)`, so a broken guard fails on the missing file instead of replacing the test binary.

`leaf_enforcement_test.go` pins the allowlist through `scankit.AssertImportAllowlist(t, "internal/selfexec", "github.com/Knatte18/loomyard/internal/logger")`, as `internal/segmentcolor` does, and `testmain_test.go` runs the package under `tmuxkit.Main` like `internal/segmentcolor`'s.
`pattern/PATTERN-leaf-packages.md` gains an `internal/selfexec` section: it imports only the standard library and `internal/logger`, the logger admitted because the helper logs its stat, exec and replace outcomes.
`cmd/lyx/testdata/test-budget.yaml` gains `internal/selfexec: 2`, in sorted position.
`docs/overview.md`'s package tree gains an `internal/selfexec/` line beside the other leaves ("the self-re-exec helper a long-lived pane process calls on a changed binary, a leaf over the standard library and `internal/logger`").

**ImpactSummary:** A new leaf package with no caller yet, one PATTERN background section, one budget row and one doc tree line; nothing existing changes behavior.
