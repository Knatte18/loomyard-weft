# Relevant code

- `internal/reedengine/strand.go` `AddStrand`, `validateDisplay`, `refuseWindowMove` — holds the one-overview refusal D1 removes
- `internal/reedengine/replace.go` `ReplaceStrand` — the replace-with-overview refusal and the select-on-replace rule
- `internal/reedengine/windoworder.go` `overviewWindowInsertArgs`, `ownWindowInsertArgs`, `overviewWindowTitle` — window placement and naming D1 changes
- `internal/reedengine/spawn.go` `markOwnWindowLocked` — opens an overview window and sets its markers
- `internal/reedengine/ownwindow.go` `isOverview`, `agentWindowPanes`, `relevantWindow` — the single-overview focus fallback D2 changes
- `internal/reedengine/follow.go` `followTargetWindow` — engine-less follow target, falls back to the overview window
- `internal/reedengine/render/types.go` `Display`, `AnchorOverview` — the display gaining the order key
- `internal/reedengine/doc.go` — the "Overview anchor" and focus-rule paragraphs
- `internal/orchcli/runs.go` `runsStrandAddSpec`, `ensureRunsStrand`, `RunsStrandName` — the runs strand's spec and the replacement D4 removes
- `internal/orchcli/start.go` `beamUp` — where `lym beam start` ensures the runs strand
- `internal/orchcli/doc.go` — states the runs strand replaces a loom status strand
- `internal/loomcli/bootstrap.go` `statusStrandAddSpec`, `resolveStatusStrandAction` — the status strand's spec and keep/replace/add choice
- `internal/loomcli/start.go` — the start path that brings reed up and ensures the status strand
- `internal/loomcli/cli.go` `loomCLI` — carries the run's recipe name
- `internal/loomcli/arm.go` `resolveRecipe` — reads the recipe the status strand's title takes
- `internal/loomcli/sharedbootstrap.go` `ensureStatusStrand`, `seedAndCommitBootstrap`, `resolveSeedRecipe` — loom start's status strand work, and a fresh start seeds the same recipe `resolveRecipe` read
- `internal/reedcli/add.go` — refuses the overview anchor on `lym reed add`, bounding D1
- `internal/battencli/wire.go` — a batten pass spawns `lym loom start --no-attach`, the heal D5 rests on
