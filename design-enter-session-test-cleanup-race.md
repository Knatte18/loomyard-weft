# TestEnterSession_DrivesTheToldTmux fails at temp dir cleanup now and then

`enter-session-test-cleanup-race` · Task · bug, reed · running · Darn-Write

`TestEnterSession_DrivesTheToldTmux` in `internal/reedcli` failed once in a tier1 gate run of `dotnet-gate-fixes` (2026-11-26, run `20261126T191602Z-tier1-1241782`), then passed 30 of 30 runs with `-count=30`.
The failure is in cleanup, not in an assertion:
`TempDir RemoveAll cleanup: unlinkat .../TestEnterSession_DrivesTheToldTmux.../001/container/warp-bare-LYMHUB/warp-bare: directory not empty`.
Something the test starts most likely still writes into its temp dir when the test returns, such as a background process or a goroutine that outlives it.

Idea: make the test wait for whatever it started to stop before it returns, so cleanup finds the directory quiet.
Each flaky failure costs a run a full red verify round, about 5 minutes, and a gate-fixer turn.


## From issue #266

Imported from [issue #266](https://github.com/Knatte18/loomyard/issues/266).

**What happened.**
`TestEnterSession_DrivesTheToldTmux` in `internal/reedcli` failed once in a tier1 gate run with:

```
TempDir RemoveAll cleanup: ... directory not empty
```

It passed in the gate runs before and after, and in 30 repeats.

**Why it is abnormal.**
The test is flaky, and the one failure cost an extra full gate run.

**What to do.**
The likely cause comes from reading the code and has not been reproduced:
the test's cleanup cancels the context of `watchPaneOOMKills` (`internal/reedcli/oomnotice.go`, started in `internal/reedcli/watchdog.go`) without waiting for the goroutine to return.
The goroutine's first strand read can then still be writing under the hub while the temp directory is removed.
Confirm this, then make the goroutine's exit awaitable and have the cleanup wait for it before the temp directory is removed.

Provenance: task `dotnet-gate-fixes`, note `gate-fixer-darn-write.md`.
