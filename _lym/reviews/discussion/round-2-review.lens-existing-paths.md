```yaml
lens: existing-paths
outcome: reported
```

**existing-paths lens: the paths each decision's state meets, and whether the decision survives each one.**

D1/D2/D3 (`board-data.js`, `board.html`, render manifest, sync, replay):
- `Board` critical-section render: `RenderToDisk` (`internal/boardengine/board.go:192`) writes every name `Render` returns. Survives.
- Owner-set write (`internal/boardengine/ownerset.go:303-308`) commits every name `Render` returns. Survives.
- Replay `Apply` (`internal/boardengine/replay.go:472`) re-renders through `RenderToDisk`, so both files are rewritten once `Claims` claims them. Survives.
- Stale removal reads the gitignored, per-machine manifest (`internal/boardengine/render.go:51-57`, `internal/boardengine/sync.go:79-111`). Only a machine whose own manifest lists the files removes them on a downgrade. Another older machine leaves them stale, as D3's bound says. Survives.
- The render timestamp in `board-data.js` makes every render change its bytes. The README already embeds a render time (`internal/boardengine/render.go:388`), so this adds no new churn class. Survives.
- `ensureBoardIgnores` ignores only the manifest, so both files are committed. Survives.

D5 (`lym board open` pre-run):
- Board `PersistentPreRunE` (`internal/boardcli/cli.go:125-175`): the other verbs keep `Resolve`, and `--board-path` bypasses resolution. Survives.
- The root pause gate (`cmd/lym/main.go:119-126`) holds only run-annotated commands, so `board open` is not held. Survives.

D6 (`tasks.json`, workspace files, keybindings block):
- `tasks.json` has one writer, `vscode.WriteConfig`, reached only from `internal/ideengine/spawn.go:265`. A tracked file is skipped (`internal/ideengine/spawn.go:246-255`), as the D6 bound states. Survives.
- Workspace files: callers are fabric verbs (`regenerateAfterVerb`), `ideengine.Spawn` (`internal/ideengine/spawn.go:120`), `internal/loomcli/pawlwiring.go:116` and `internal/fabricengine/anchorchange.go:539`. All run in a lym process where the root pre-run installs process-wide hooks (`cmd/lym/main.go:158-182`). Survives.
- The hook is read outside the hub workspace lock, as the namer is. Nothing in the ide hook takes a lock. Survives.
- The beam mark is prepended before `BuildWorkspace`'s `{` (`internal/fabricengine/hubworkspace.go:227-229`), and `carriesBeamMark` parses only the head (`internal/fabricengine/hubworkspace_write.go:96-107`). A `tasks` member spliced after `settings` disturbs neither. Survives.
- `removeStaleBeamWorkspaces` reads only the mark. Survives.
- Keybindings: `SeedKeybindings` is called only from `internal/ideengine/spawn.go:242`. The merge replaces the whole lym block in place (`internal/vscode/keybindings.go:208-214`), so the two new entries ride it. Survives.

D4, D7: no existing writer or reader beyond the render paths above.

Findings: none. No existing path undoes or misreads a decision where the record is silent.

Tool calls:
1. Render manifest, gitignore and workspace writer and stale-removal code: the manifest only named `RenderToDisk`, not the files it touches.
2. Every `tasks.json`, `SeedKeybindings` and workspace-file reference, plus `vscode.BuildWorkspace`: needed every other writer and reader of these files.
3. `RenderToDisk` and `FabricReplay.Claims`: needed the manifest removal logic and the claim set.
4. Replay `Apply` and every render caller: needed whether replay re-renders all outputs.
5. Render timestamp in the README: needed whether a timestamp in `board-data.js` adds new churn.
6. `installPauseGate`: needed whether a pause gate holds the new verb.
