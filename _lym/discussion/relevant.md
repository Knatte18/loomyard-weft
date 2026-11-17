# Relevant files

- `internal/boardengine/render.go` `RenderToDisk`, `Render`, `readRenderManifest`, `writeRenderManifest`, `layerSection`, `notesTypeOrder` — the render the two new outputs and the collapsible sections join
- `internal/boardengine/replay.go` `FabricReplay.Claims`, `isBoardOverview` — the replay claim the new file names join
- `internal/boardengine/labels.go` `validateLabels` — how type labels are told from area labels through the vocabulary
- `internal/boardengine/config.go` `Outputs`, `Config.Outputs` — the render inputs the projection reads
- `internal/fabricengine/bolt.go` `Bolt.Commit`, `Bolt.CommitWritten` — how a board write's rendered files are committed
- `internal/fabricengine/boltreplay.go` `claimantOf` — replay drops paths no replayer claims
- `internal/fabricengine/junctionnames.go` `BoltDir` — the hub's Bolt path the open verb resolves
- `internal/boardcli/cli.go` — the board cobra subtree the open verb joins, beside `rerender`
- `internal/ideengine/spawn.go` `seed`, `isBeam`, `writeVSCodeConfig`, `seedKeybindings` — where the prime, beam or task classification and the VS Code writes happen
- `internal/vscode/config.go` `WriteConfig` — writes `tasks.json`, where the two tasks join `reed attach`
- `internal/vscode/keybindings.go` — the `// lym:begin` block writer the two keys join
- `internal/beamcli/start.go` `ownedTargets` — `lym beam start` acts on a prime or beam worktree
- `internal/fabricengine/hubworkspace.go` `RegenerateHubWorkspace`, `BeamWorkspacePath` — writes the hub and beam workspace files the `beam start` task joins
- `internal/boardengine/board.go` `Board.Rerender` — the refusal's way forward renders and syncs
