```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1, F3 and F4 and check the fix covers every member.

**F1 (unreaped `cmd.Start()` sites): covered.**
- I listed every non-test `.Start()` call under `internal` and `cmd` and matched each against `proc.Reap(` or `.Wait()`.
- Every site now waits or reaps: `lifecycle.go`, `launch_linux.go` and `launch_windows.go` (this round), plus the other sites with a wait or reap (`spawnwatchdog.go`, `reedcli/remove.go`, `termwindow.go`, `fronttask/hub.go`, `boardengine/spawn.go`, `adoptcli/deps.go`, `battencli/cli.go`, `dobbyengine/start.go`, `fabricengine/spawn.go`).
- The `.Wait()` sites include `beamcli/dobbydeps.go`, `creelcli/spawn.go`, `orchcli/spawn.go`, `gatecli/detachfront.go`, `loomcli/drivewiring.go`, `driftengine/spawn.go`, `shedverbs/loopwait.go`, `verifyrun/run.go` and `fabricengine/remoteteardown.go`.
- `proc/tied_*.go` and `cmd/testtiming` also wait.
- No unwaited start remains.

**F4 (stale "never waited" comments): covered.**
- The grep for "never wait", "not wait", "never Waited" and "never reaped" shows no remaining comment that contradicts a reap.
- The hits it did return are unrelated "returns without waiting on a remote or lock" comments.

**F3 (`lister` callers): covered.**
- `lister{` is built only in `record.go:120` and `end.go:139` in production code, and in the two `record_test.go` cases.
- All four set both `isAlive` and `isRunning`.
- The old `live` method has no remaining caller.

Findings: no findings.

Unchecked: F2, F5, F6 and F7 are not set-premise findings, so I did not search for them.

Tool calls:
1. Grep of `.Start()` and `proc.Reap(` / `.Wait()` call sites across `internal` and `cmd` (one Bash call): the review's F1 set needed re-running and my context held only the named sites.
2. Grep for stale "never waited" comments and for `lister{` / `.live(` usage (one Bash call): the F4 and F3 sets needed re-running.
