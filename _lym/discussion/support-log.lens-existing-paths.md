```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every code path that already writes, reads, commits or replays the state D1–D7 add or change.

**Findings**

1. LOW, D3 (replay of a render-only commit). `FabricReplay.Apply` sorts claimed files into board.json, design, record and raddle-store buckets. Overview files are claimed but never bucketed (`replay.go` ~352–361), and the function re-renders only when the result is not Empty (~462–474).
   - A local commit that changes only rendered files is replayed as Empty. Examples: `lym board rerender` after an upgrade, or a new-viewer `board.html` carried by a write whose `board.json` change is refused.
   - That drops the new `board.html` until the next board write.
   - It self-heals. But D2 says "the next board write after that upgrade commits it", and on a diverged Bolt that is only true once some later write is not Empty. The record does not say so. Evidence: `run.result.Empty = !run.applied && !designsApplied` returns before `RenderToDisk`.

2. LOW, D2 (version skew across machines). The render manifest is per machine: `.board-rendered.json` is gitignored.
   - **Older binary.** A machine on an older lym never writes the two files. Each board write it makes leaves `board-data.js` stale against `board.json` until a newer machine writes.
   - **Downgrade.** A binary rolled back on a machine whose manifest lists the new names deletes both files, since `RenderToDisk` removes previous-manifest names it did not produce. The sync then commits the deletion.
   - **Two new binaries with different viewers** each rewrite `board.html` on every write.
   - The README format has the same exposure. Still, D2's claim that the file "changes … only when a new lym binary ships a changed viewer" holds only with one lym version per hub. The record does not say so.

3. LOW, D6 (`tasks.json` writes skipped and stale worktrees).
   - `ideengine.writeVSCodeConfig` skips the write when `tasks.json` is git-tracked. A repo that commits `.vscode/tasks.json` gets neither new task, while the user-global keys still name them.
   - A worktree seeded before the upgrade keeps the one-task `tasks.json` until its next seed or spawn. Meanwhile the keybindings block is rewritten by the first seed of any worktree, so `alt+shift+9` misses in older windows.
   - The bound under D6 covers only the task-pair case of `alt+shift+0`.

**Paths traced, per decision**

- **D1, D2 and D3 (render outputs):**
  - `Board.boardCriticalSection` runs Save, then `RenderToDisk`, then `spawnSync`: survives. The files land in the same sync.
  - The sync's `commitDirty` commits through `Bolt.Commit`, which reaches `commitWeftAt`, which calls `StageAllAndCommit`. It stages the whole tree with no pathspec, so the new files commit beside `README.md`: survives.
  - `ensureBoardIgnores` ignores only `.board-rendered.json`, so neither file is hidden: survives.
  - `renderOwnerWrite` with `Bolt.CommitWritten` commits the names in `Render`'s map plus removed previous names. Both new names join automatically: survives.
  - The stale-file removal in `RenderToDisk` reads the manifest: survives, with the downgrade caveat in finding 2.
  - `FabricReplay.Claims` with `claimantOf` in fabric drops unclaimed paths, and D3 adds the claim: survives.
  - `FabricReplay.Apply` ignores overview files and re-renders after a non-empty replay: survives, with the gap in finding 1.
  - `withMissingRecordsUndiscussed` clears the discussed flag when the decision file is missing, so the projection's decision file name stays consistent: survives.
- **D4 (viewer):** It reads only `window.BOARD`, and no other code reads `board-data.js`: survives.
- **D5 (`lym board open`):** It reads `board.html`, which only `RenderToDisk` writes and removes. The refusal path covers the case before a first render: survives.
- **D6 (VS Code tasks and keys):**
  - `seed` calls `writeWorktreeConfig` before it classifies the worktree. The record's plan note covers the reorder: survives.
  - `isBeam` reads an unreadable record as a task, which D6 states: survives.
  - `seedKeybindings` and `mergeKeybindings` rewrite the whole block on seed and on spawn, so the new keys are rewritten with it: survives.
  - The skipped write for a git-tracked `tasks.json` and the pre-upgrade worktrees are not covered: finding 3.
- **D7 (collapsible sections):** No Go code outside `boardengine`'s `render.go`, `replay.go` and `doc.go` names `README.md`, `project-*.md` or the section headings. The only parse is `isBoardOverview`, by file name, so nothing reads the changed section markup: survives.

**Tool calls made**

- `grep` for `RenderToDisk`, `isBoardOverview` and `overviewFile` in boardengine, plus `replay.go` 440–500: I needed every render caller and the replay's re-render condition.
- `grep` for `renderOwnerWrite` in `ownerset.go`: I needed what the owner-set path commits.
- `grep` for how `Apply` sorts files, then a Read of `replay.go` 345–374: I needed what replay does with claimed overview files.
- `grep` for other readers of `README.md`, `project-*.md` and the section headings, and for a Bolt-root file allowlist: I needed to know whether anything else reads the rendered markup.
- `grep` for `commitWeftAt`: I needed whether the sync commit stages the whole Bolt or a pathspec.
