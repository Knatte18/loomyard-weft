```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code that writes, rewrites or reads each file or setting the decisions add or change.

**Paths traced, per decision**

- **D1 / D2 (render outputs):**
  - `boardengine.RenderToDisk` writes each output and lists it in the manifest, so stale-file removal works (`internal/boardengine/render.go:40-60`). Survives.
  - Board sync goes `commitDirty` → `bolt.Commit` → `commitWeftAt`, which stages the whole Bolt tree (`internal/boardengine/sync.go:72`, `internal/fabricengine/bolt.go:36`). Survives.
  - `ensureBoardIgnores` adds only `.board-rendered.json` to the ignore file, so neither new file is ignored (`internal/boardengine/sync.go:81-106`). Survives.
  - No code lists or sweeps unknown files in the Bolt root; my grep for `ReadDir` on the board dir and for overview names found no board match. Survives.
- **D3 (commit and replay):**
  - The owner-set path: `renderOwnerWrite` returns every name in `Render`'s map plus the removed names, and `CommitWritten` commits exactly those paths (`internal/boardengine/ownerset.go:296-321`, `internal/fabricengine/bolt.go:49-68`). Survives; the record's claim holds.
  - Replay: `claimantOf` drops any path no replayer claims (`internal/fabricengine/boltreplay.go:147`). `FabricReplay.Apply` ignores claimed files that are not design, record or store files, and re-renders through `RenderToDisk` (`internal/boardengine/replay.go:342-475`). Survives, with the empty-replay bound D3 already states.
- **D6, workspace `tasks` key:**
  - A new top-level key sits after `lym.generated`; `markBeamWorkspace` keeps that mark first and `carriesBeamMark` reads only the head of the file (`internal/fabricengine/hubworkspace.go:227-230`, `internal/fabricengine/hubworkspace_write.go:96-107`). Survives.
  - `removeStaleBeamWorkspaces` checks only that mark. Survives.
  - `writeHubWorkspace` rewrites a file only when its bytes change. Survives.
  - The anchor-change path `AnchorChange.wireAtTarget` regenerates the files at the new anchor, so the task's working directory follows a moved anchor (`internal/fabricengine/anchorchange.go:533-540`). Survives.
  - Every regenerating caller rewrites the `beam start` tasks: `regenerateAfterVerb` after add, remove, clone, prune, reconcile and checkout; `ideengine.Spawn`; the beam-mark step in `internal/loomcli/pawlwiring.go:111-118`; and `AnchorChange`. Under mixed lym versions this **breaks**; see finding 1.
- **D6, `tasks.json`:**
  - `vscode.WriteConfig` overwrites the whole file on every seed or spawn (`internal/vscode/config.go:62-83`).
  - The tracked-file skip is in `internal/ideengine/spawn.go:246-256`; D6's bound already names that case. Survives.
  - An older binary's seed or spawn rewrites the file with no `board open` task. **Breaks**; see finding 1.
- **D6, keybindings block:**
  - Every `lym ide seed` or `lym ide spawn` rewrites the whole `// lym:begin` block from `keybindingsBlockEntries` (`internal/vscode/keybindings.go:34-40`).
  - An older binary's rewrite drops `alt+shift+0` and `alt+shift+9`. **Breaks**; see finding 1.
- **D7 (collapsible sections):**
  - No code parses the README's section structure. Only `boardengine` reads it, and the existing `<details open>` per project is at `internal/boardengine/render.go:529`. Survives.
- **D5 (`lym board open`):**
  - It writes no state, and it reaches `board.html` only through the render paths above. Nothing else to trace.

**Findings**

1. MEDIUM · design · D6 (its Bound paragraph)
   - **Summary:** Every hub and beam workspace file, `tasks.json` and the user keybindings block is regenerated whole by whichever lym binary acts last. A machine or process still on an older lym therefore strips the `beam start` workspace tasks, the `board open` task and both keys, while the user-global keys can stay behind and name missing tasks.
   - **Triggers for the workspace files:** any fabric verb that calls `regenerateAfterVerb`, the beam-mark step in pawlwiring, or an anchor change, run by the older binary.
   - **Triggers for `tasks.json` and the keybindings block:** an older binary's `lym ide seed` or `lym ide spawn`.
   - **Gap in the record:** D6's bound names only windows seeded before the upgrade. D3 states the same mixed-version exposure for the board files, but D6 says nothing about it for these files.
   - **Evidence:**
     - `internal/fabricengine/hubworkspace.go:130-134` and its callers in `add.go:471`, `remove.go:264`, `clone.go:412`, `prune.go:95`, `reconcile.go:276` and `checkout.go:97`
     - `internal/fabricengine/anchorchange.go:539`
     - `internal/loomcli/pawlwiring.go:116`
     - `internal/vscode/config.go:62-83`
     - `internal/vscode/keybindings.go:34-40`
2. LOW · design · D6 (the `beam start` task's command)
   - **Summary:** Each regenerating process stamps its own binary into the workspace files. The callers are fabric verbs, loom's pawl wiring and the anchor change, each running whatever binary started it. `selfexec.LymBinary` returns `ErrTestBinary` under go test, and `ideengine` handles that by dropping the error and falling back to the bare `lym` (`ideengine/spawn.go:231`, `vscode/config.go:71-73`). D6 does not say whether `beam start` gets the same fallback, and it does not bound the binary path a regeneration may stamp into the file, so the task's command depends on the last regenerating process.
   - **Evidence:** `internal/selfexec/binary.go:19-40`, `internal/ideengine/spawn.go:231`, `internal/vscode/config.go:71-73`

**Tool calls**

1. Board sync commit and `ensureBoardIgnores` in `internal/boardengine/sync.go`: to learn how the sync stages files and what it ignores; my context did not have sync.go.
2. `Bolt.Commit`, `Bolt.CommitWritten` and `classifyPaths`: to check what a board write commits; my context did not have bolt.go or classify.go.
3. A grep for code that reads or sweeps the Bolt root and for `RetireLegacy` removals: to find any path that would remove the new files.
4. A grep for other workspace-file writers and readers, plus `hubworkspace_write.go` (`writeHubWorkspace`, `removeStaleBeamWorkspaces`, `carriesBeamMark`): my context did not have that file.
5. The call sites in `internal/loomcli/pawlwiring.go` and `internal/fabricengine/anchorchange.go`, plus the `regenerateAfterVerb` callers: to see which processes regenerate the workspace files.
6. A grep for README and `<details>` readers: to check whether D7 breaks any code that parses the README.
