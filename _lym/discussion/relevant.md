# Relevant code

- `internal/orchcli/runs.go` `ensureRunsStrand`, `runsStrandAddSpec`, `RunsStrandName` — takes the overview window from the loom status strand; D1 changes it
- `internal/loomcli/bootstrap.go` `statusStrandAddSpec`, `statusStrandCmd` — the status strand's spec and command that D2 makes one shared builder
- `internal/loomcli/sharedbootstrap.go` `ensureStatusStrand` — loom start's keep, replace or add of the status strand and its sidecar write
- `internal/loomcli/statusbuild.go` `statusSidecarPath` — the status strand's sidecar path D1 must also write
- `internal/loomcli/pawlwiring.go` `pawlDeps`, `runBeamUpStep` — the pawl row that calls `BeamUp` after the run's loom start
- `internal/reedengine/replace.go` `ReplaceStrand` — removes a strand and adds any spec in its place
- `internal/reedengine/render/types.go` `Display` — `Anchor`, `KeepViewer`, `Window` and `Title`, which place and name a strand's window
- `internal/reedengine/ownwindow.go` `planOwnWindowPlacement`, `newAgentWindowSelected` — when a window group gets a window named for it, and that `KeepViewer` never takes the viewer
- `internal/fabriccli/anchordeps.go` `runStatusUnfinished` — the existing done-means-finished test D1 and D3 match
- `internal/shedengine/status.go` `StateDone` — the one state D1 and D3 treat as finished
- `internal/loomengine/driver.go` `LoomStatusStrandName` — the status strand's fixed role
- `internal/battencli/overview.go` `overviewStage`, `overviewSource.frame`, `DobbyRunCells` — the beam overview D3 adds a line to
- `internal/shedrun/seed.go` `ReadSeed` — reads the self run's recipe
- `internal/shedrun/paths.go` `StatusFile` — the self run's status file path
- `internal/shedrun/runid.go` `SelfRunID` — the worktree's own run id
