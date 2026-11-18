# Relevant files

- `internal/orchcli/runs.go` `ensureRunsStrand`, `runsStrandAddSpec`, `RunsStrandName` — replaces the checklist strand today; gains the move
- `internal/orchcli/start.go` `beamUp` — calls ensureRunsStrand and carries its warning on the report
- `internal/orchcli/doc.go` — states the overview-window rule the change revises
- `internal/loomcli/bootstrap.go` `statusStrandAddSpec`, `statusStrandCmd`, `resolveStatusStrandAction` — the checklist spec, command and keep rule to share
- `internal/loomcli/sharedbootstrap.go` `ensureStatusStrand` — loom start's keep, replace or add of the checklist strand and its sidecar write
- `internal/loomcli/statusbuild.go` `statusSidecarPath` — the sidecar path orchcli must also write
- `internal/loomcli/pawlwiring.go` `pawlDeps`, `runBeamUpStep` — the Beam-Up row that runs BeamUp inside the pawl run
- `internal/loomengine/driver.go` `LoomStatusStrandName` — the pinned checklist role; home of the shared spec
- `internal/reedengine/render/types.go` `Display` — Title is read only for an overview strand today
- `internal/reedengine/spawn.go` `overviewWindowTitle` — names a new window by overview title or role
- `internal/reedengine/ownwindow.go` `planOwnWindowPlacement` — own-window placement and the break-out from the shell window
- `internal/reedengine/lifecycle.go` `StrandStatus` — carries no command or display
- `internal/battencli/overview.go` `overviewSource.frame`, `DobbyRunCells.poll` — the runs strand's frame and its seed and status readers
- `internal/shedrun/seed.go` `ReadSeed` — reads the run's recipe
- `internal/statuspane/build.go` `WriteSidecar`, `ReadSidecar`, `CurrentBuild` — the build sidecar both paths write
