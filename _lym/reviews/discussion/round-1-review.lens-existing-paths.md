```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: I traced every existing code path that writes, reapplies or reads the state the record's decisions introduce or change.

**Paths traced, per decision**

- D1/D3, `boardengine.RenderToDisk` (`internal/boardengine/render.go:43-70`): the write loop and manifest-driven stale removal pick up any new name `Render` returns. Survives.
- D1/D8, `RenderToDisk` passes `Render` the `withMissingRecordsUndiscussed` copy, which clears `Discussed` on an entry whose decision file is missing (`render.go:44`, `render.go:73-85`). The record does not say whether the projection and its per-entry revision are built over that copy or over the stored entry. Breaks: finding 1.
- D3, owner-set write: `renderOwnerWrite` commits every name `Render` returns plus removed manifest names (`internal/boardengine/ownerset.go:324-349`). Survives.
- D3, Bolt replay: `FabricReplay.Apply` re-renders through `RenderToDisk` after applying (`internal/boardengine/replay.go:613`), so both new files are re-rendered once claimed. An empty replay returns before the render (`replay.go:603-606`), which matches D3's bound. Survives.
- D3, downgrade: an older binary's `RenderToDisk` removes the names in the manifest that it no longer renders (`render.go:61-67`), as D3 states. Survives.
- D3, `lym board rerender` runs the same `RenderToDisk`. Survives.
- D8, Bolt replay writes `board.json` without taking the lock itself; its caller holds the board and push locks (`replay.go:460`). D8's claim that the check is atomic against every other writer holds. Survives.
- D8, sync commit `commitDirty` takes the same write lock (`internal/boardengine/sync.go:59-77`). Survives.
- D8, the write section's load runs `migrateDraftStatus` and then `migrateDiscussedStatus` (`internal/boardengine/board.go:116-117`), and D8 hashes after them. Survives.
- D8, `Task` JSON (`internal/boardengine/task.go:19-79`): slices without `omitempty`, a `Status` pointer, and `Unknown` written back in key order. A load-then-encode round trip is stable. Survives.
- D11, `.lym` under the Bolt: `bolt.Commit` keeps never-committed dirs out, and creel's journal and lock already sit there. Survives.
- D11, takeover: `runproc.EndSubject` for `HubOwner` ends every role except the ones in `subjectPassSpared` (dobby, teardown, adopt, sync, push, slack-relay) (`internal/runproc/end.go:97-100`). D11's claim holds only for a role outside that list, and the record names no role. Finding 2.
- D6, `vscode.WriteConfig` overwrites the whole `tasks.json` on every seed (`internal/vscode/config.go:60-83`, `config.go:177-202`), so the new `board open` task is rewritten with it. Survives.
- D6, `writeVSCodeConfig` skips a worktree with a git-tracked `tasks.json` (`internal/ideengine/spawn.go:247-258`), matching D6's bound. Survives.
- D6, `vscode.SeedKeybindings` rewrites the whole block from `keybindingsBlockEntries` (`internal/vscode/keybindings.go:34-40`). Survives.
- D6, every caller of `RegenerateHubWorkspace` runs inside the lym binary, where `cmd/lym/main.go` installs the hook: ideengine Seed and Spawn, fabric's `regenerateAfterVerb`, `anchorchange.go:545`, `beamcli/dobbydeps.go:427`, `loomcli/pawlwiring.go:120` and `adoptcli/deps.go:110`. Survives.
- D6, `markBeamWorkspace` strips only the opening `{\n` (`internal/fabricengine/hubworkspace.go:227-230`), and `carriesBeamMark` reads only the first key, so members spliced after `settings` leave both intact. Survives.

**Findings**

1. **MEDIUM, design.** Location: `_lym/discussion/decision-record.md` D1 and D8.
   - Every render goes through `RenderToDisk`, which hands `Render` a copy with `Discussed` cleared for an entry whose decision file is missing. D1 says the projection, revision included, is "computed by the same code that builds the README". D8 says the revision hashes the entry as the write section holds it, with `Discussed` still true.
   - If the projection's revision is computed inside the render path over that copy, the page holds a revision the check never computes, and every edit of such an entry is refused as stale.
   - If the server instead builds the projection without the copy, it lists a decision file name that does not exist: `/file/&lt;name&gt;` fails and the `file://` link is broken.
   - The record should state that the revision is taken over the stored entry before the render's copy, and that the projection's decision-file names follow the copy.
   - Evidence: `internal/boardengine/render.go:43-44`, `render.go:73-85`, `internal/boardengine/board.go:116-118`.

2. **LOW, design.** Location: `_lym/discussion/decision-record.md` D11 ("A hub takeover by another machine ends it … through `runproc`") and D13.
   - Whether a takeover ends the server depends on its `runproc` role. `EndSubject` spares the roles that serve the whole hub (sync, push, slack-relay, adopt, dobby, teardown).
   - D11 registers the server with `runproc.SelfRecord` under `HubOwner` but names no role. D13 lists no `runproc` change, although PATTERN-child-processes asks for "a role the package names".
   - Evidence: `internal/runproc/end.go:95-101`, `internal/runproc/record.go:29-55`.

**Tool calls**

1. `render.go:40-115` and `renderOwnerWrite`: the manifest held no `RenderToDisk` body or owner-set render, which D3 and D8 depend on.
2. Callers of `BuildWorkspace`, `WriteConfig`, `RegenerateHubWorkspace` and the keybinding writers, plus replay's render: to find the paths that reapply D6's members and D3's files.
3. `vscode/config.go` `WriteConfig` and `writeAttachTask`, and the keybindings block entries: to check that the D6 writers rewrite whole files.
4. `ideengine/spawn.go` `writeVSCodeConfig` and the `replay.go` Apply render tail: to check D6's tracked-file skip and D3's replay re-render.
5. Lock use in `replay.go` and the `writeLockFile` users: to check D8's claim that the check is atomic against the replay writer.
