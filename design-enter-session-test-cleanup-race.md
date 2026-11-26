# TestEnterSession_DrivesTheToldTmux fails at temp dir cleanup now and then

`enter-session-test-cleanup-race` · Note · bug, reed

`TestEnterSession_DrivesTheToldTmux` in `internal/reedcli` failed once in a tier1 gate run of `dotnet-gate-fixes` (2026-11-26, run `20261126T191602Z-tier1-1241782`), then passed 30 of 30 runs with `-count=30`.
The failure is in cleanup, not in an assertion:
`TempDir RemoveAll cleanup: unlinkat .../TestEnterSession_DrivesTheToldTmux.../001/container/warp-bare-LYMHUB/warp-bare: directory not empty`.
Something the test starts most likely still writes into its temp dir when the test returns, such as a background process or a goroutine that outlives it.

Idea: make the test wait for whatever it started to stop before it returns, so cleanup finds the directory quiet.
Each flaky failure costs a run a full red verify round, about 5 minutes, and a gate-fixer turn.
