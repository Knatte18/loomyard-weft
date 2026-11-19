```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the paths that already write or read what each decision introduces or changes. None breaks, so I have no findings.

Paths traced, per decision:

- **D1/D3, the board's own write.** `boardCriticalSection` renders through `RenderToDisk` (`internal/boardengine/board.go:192`), so both new files are written and listed in the manifest. **Survives.**
- **D3, the owner-set write.** `renderOwnerWrite` commits every name `Render` returns, plus the previous manifest's removed names (`internal/boardengine/ownerset.go:324-349`). The new files ride along. **Survives.**
- **D3, a Bolt replay.** `FabricReplay.Apply` re-renders through `RenderToDisk` (`internal/boardengine/replay.go:613`), and `Claims` is extended per D3. **Survives.**
- **D3, a manifest removal.** On a downgrade, the stale-file removal in `RenderToDisk` (`internal/boardengine/render.go:76-82`) removes both files. D3's bound already states this. **Survives.**
- **D8, a replay or pull rewriting `board.json`.** `Bolt.Pull` takes the board write lock before `pullFirst` (`internal/fabricengine/bolt.go:279-283`). That is the same lock `boardCriticalSection` takes (`internal/boardengine/sync.go:23`, `board.go:97`). So D8's compare and write stay atomic against a replay. **Survives.**
- **D8, migrations on load.** `migrateDraftStatus` and `migrateDiscussedStatus` run under the lock before the mutation (`internal/boardengine/board.go:117-118`). D8 hashes after them. **Survives.**
- **D11, the state directory under the board-level `.lym`, against the sync's stage-all.**
  - A one-repository Bolt seeds `/.lym` (`internal/fabricengine/recordsgit.go:420-425`).
  - Any other weft seeds `.lym/` (`recordsgit.go:120`).
  - The token never reaches a commit in either hub shape. **Survives.**
- **D11, a hub takeover.** `EndSubject` spares only the roles in `subjectPassSpared` (`internal/runproc/end.go:100`). The new role is kept out of that list, so the server is ended. **Survives.**
- **D5/D11, the liveness read.** It follows the same try-and-release shape as `dobbyengine.Live` (`internal/dobbyengine/start.go:273-285`). **Survives.**
- **D6, seed and spawn rewriting `tasks.json`.**
  - `WriteConfig` overwrites the whole file (`internal/vscode/config.go:70-91`), so `board open` is rewritten on every seed.
  - A tracked `tasks.json` is skipped (`internal/ideengine/spawn.go:246-255`). D6's bound already states this. **Survives.**
- **D6, the workspace regeneration and stale-beam removal.**
  - `carriesBeamMark` reads only the first key (`internal/fabricengine/hubworkspace_write.go:94-102`).
  - Members spliced after `settings` leave the mark first.
  - No other production reader of `.code-workspace` content exists. **Survives.**
- **D6, the keybindings block rewrite.** `keybindingsBlockEntries` is rewritten whole (`internal/vscode/keybindings.go:33-39`). The new keys join it, and `ctrl+alt+a` is kept. **Survives.**
- **D10 file route, decision and design files.** These are written by board writes and replays. The route reads them only for a name the current projection lists. **Survives.**

Tool calls:
1. `internal/boardengine/ownerset.go` 300-370, `recordsgit.go` 405-440, and a grep of render callers: the manifest lacked the owner-set commit list, the Bolt `.lym` exclude and every `RenderToDisk` caller.
2. A grep for `tasks.json` writers and the seed's tracked-file skip: the manifest lacked any other writer of `tasks.json`.
3. `seedWeftArtifactExcludes`, workspace-file readers, and `pullFirst`'s locks: the manifest lacked the exclude for a hub that is not one repository, readers of the workspace file, and the replay's lock.
4. `internal/fabricengine/bolt.go` 270-302 and the `BoltLockPath` callers: these confirm that a replay holds the same board write lock D8 relies on.
