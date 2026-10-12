# Card 5 — gate-prebuild-run-directory

**Edit:**
- `internal/gatecli#prebuildLyx`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `cmd/lyx#main`

**Uses:**
- `internal/lyxtemp#MkdirTemp`
- `internal/lyxtemp#Dir`
- `internal/lyxtemp#Remove`

**Intent:**
`lyx gate test`'s prebuilt-`lyx` directory is created under the process run directory through `lyxtemp.MkdirTemp("lyx-prebuilt-")` and keeps its existing cleanup, which removes it after go test ends (D5).
`cmd/lyx`'s `main` calls `lyxtemp.Remove()` after `notifyExitAndSweep(code)` and before `os.Exit(code)`, so a `lyx` process that created a run directory removes it on normal exit; a process that never created one removes nothing, and a removal error is ignored there, since the next start's sweep finishes it.
The testable `run` seam does not call it: there the process is a test binary whose `tmuxkit.Main` owns the run directory.
Bound: after a SIGKILL of `lyx gate test`, the prebuilt binary stays until a later start's sweep, at least 10 minutes on; its package children already die with it through the gate's parent-death signal.
That reclaims the 34 MB binary a killed gate run leaves today.

`TestGateTest_Scenario`'s tagged-run step, which already compares the prebuilt path the stand-in saw with the built binary, also asserts that path lies under `lyxtemp.Dir()`, the test process's own run directory, since the gate runs in-process there.
`internal/gatecli/doc.go`'s "<tmp> is a per-invocation temporary directory removed after go test ends" says the directory sits in the process run directory.

**ImpactSummary:** The gate's prebuilt binary moves into the process run directory and lyx's main removes that directory on exit; no caller signature changes.
