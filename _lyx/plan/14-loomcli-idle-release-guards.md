# Card 14 — The loop's idle-release wake hook logs a repeating read failure once, and a release at Info

**Create:**
- `plan:internal/loomcli#TestIdleReleaseLogLevels` -> `func TestIdleReleaseLogLevels(t *testing.T)`

**Edit:**
- `internal/loomcli#ownedLockBranches` -> `func ownedLockBranches(locks *parentlock.Store, self parentlock.Owner) ([]string, error)`
- `internal/loomcli#loopLockExit`
- `internal/loomcli#idleReleaseOn`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D11 for `internal/loomcli`: the sites the audit in `00-overview.md` lists for this package as guard or demote.

`arm.go` sets `spec.Loop.OnWake` to `loopIdleRelease`, which `shedverbs`' child watch runs on every wake, so `idleReleaseOn`'s hook body runs once per wake for the life of the loop.
While the parent lock store cannot be listed, or the agents' states cannot be read, it logged a Warn on every wake.

`ownedLockBranches` returns the list error instead of logging it, with no branches.
`loopLockExit`, which runs once per loop exit, logs `loomcli: could not list the parent lock records` at Warn with the error as `cause` and returns, as it does today.
`idleReleaseOn` holds two `logger.Episode` locals beside its `watch`, captured by the hook it returns and so living as long as the loop (the hook runs on the child watch's one goroutine):
a list error calls `listFailures.Fail("loomcli: could not list the parent lock records", "cause", err)` and resets the watch as an empty list does today, and a list without an error calls `listFailures.Recover("loomcli: the parent lock records list again")`;
an agent read error calls `agentFailures.Fail` with `loomcli: could not read the agents' states for the parent lock's idle rule` and its fields, and a read without an error calls `agentFailures.Recover("loomcli: the agents' states read again for the parent lock's idle rule")`.
A pause-covered wake reads neither and touches neither episode.
`loomcli: released the parent lock after every agent waited past the idle bound` moves from Warn to Info: a release is the idle rule working, a state change and no failure.
`loomcli: could not release the idle parent lock` stays Warn, since every release attempt resets the idle watch, so an attempt comes at most once per idle bound.
No release or keep decision changes.

`TestIdleReleaseLogLevels`, in `looplock_test.go`, is serial (it captures the process-global logger with `logcapture.CaptureVerbose` plus `logger.SetVerbosity(2)`) and table-driven over `idleReleaseOn` on `lockedStore(t, loopOwner)` with an injected awake clock, three rows:
- "a list failure warns once and logs its recovery": the row points the store's `Dir` at a regular file under `t.TempDir()`, wakes three times at time 0, restores `Dir` and wakes once more, asserting exactly one `level=WARN` and two `level=DEBUG` `could not list the parent lock records` lines and one `level=INFO` `the parent lock records list again` line, and that the record is still held;
- "an agent read failure warns once and logs its recovery": the agents func errors on its first three calls and then answers one waiting agent, with four wakes at time 0, asserting exactly one `level=WARN` and two `level=DEBUG` `could not read the agents' states` lines and one `level=INFO` `read again for the parent lock's idle rule` line, and that the record is still held;
- "a release past the bound logs at Info": one waiting agent and wakes at 0 and the bound, asserting one `level=INFO` `released the parent lock after every agent waited past the idle bound` line, no `level=WARN` line, and that the record is gone.
`TestLoopLockExit` keeps passing unchanged, since `loopLockExit`'s decisions are unchanged.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/loomcli` row rises from 222 to 223.

**ImpactSummary:** one unexported helper re-signed, with both its callers listed here, and one hook body's logging in `internal/loomcli`; no parent lock release, hand-off or keep decision changes.

**Commit:** 14: loomcli: log a repeating idle-release read failure once per episode and a release at Info
