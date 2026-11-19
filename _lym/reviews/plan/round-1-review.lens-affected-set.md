```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, key, string, file shape or behaviour a card changes, I list what reads or asserts it and which cards cover it.

**Per change: what reads it, and which cards cover it**

1. **`Render`'s file map gains `board-data.js` (card 1) and `board.html` (card 3).**
   - `RenderToDisk` and the manifest: cards 1 and 3.
   - `renderOwnerWrite`'s path list (ownerset.go:324-350) and its callers in ownerset.go, holdwrite.go and board.go:241: the plan needs no edit; card 1's ImpactSummary names it.
   - `TestRenderToDiskManifestCleanup`: cards 1 and 3.
   - The loop at render_test.go:1187 (README and project-root): not affected.
   - boardengine `doc.go` `# README`: cards 1 and 3.
   - **Missed:** the render.go file header (render.go:1-2), which says "Those are README.md, a project-&lt;slug&gt;.md … and a design-*.md". No card lists it. → F1
2. **`FabricReplay.Claims` claims the two files (card 3).**
   - `Claims` and `TestFabricReplay_Ledger`: card 3.
   - `Apply`'s file dispatch (replay.go:471-480) lets an unrecognised claimed path fall through, and the re-render rebuilds the files. `CommitRuns` (replay.go:150-158) gives them no entry. Neither needs a change.
   - boardengine `doc.go` `# Replay`: card 3.
   - **Missed:** the comment over the replay's file-name constants (replay.go:31). It lists what the replay recognises at the Bolt root: the overview, design and decision files. No card lists it. → F1
3. **`Outputs` gains `Labels` (card 1).** `Config.Outputs` and `TestOutputs`: card 1. Covered.
4. **`BuildWorkspace` is re-signed (card 8).** Its only callers are hubworkspace.go:178 and 203 plus workspace_test.go:45, all on card 8. Covered.
5. **The worktree `tasks.json` gains `board open` (card 7).**
   - `assertAttachTaskOnly` callers: spawn_clean:111 and :137, spawn_scenario:140, spawn_hub:273 (`checkTaskSlugOpensBareFolder`). All are on card 7.
   - The vscode tests: card 7.
   - The `docs/overview.md` lines at 582-587: card 7.
6. **The keybindings block gains two keys (card 7).** `TestKeybindingsBlock`, `TestSeedKeybindings`, `keybindings.go`'s header and `docs/overview.md`:608: card 7. Covered.
7. **The workspace files gain a `tasks` member (cards 8 and 9).**
   - `TestRegenerateHubWorkspace`: card 8.
   - `checkPrimeWritesHubWorkspace` and `checkBeamOpensBeamWorkspace`: card 9.
   - `TestPersistentPreRunE_InstallsSeams` (prerunseams_test.go:55): card 9.
   - The overview's workspace passage (around line 598): card 9. Covered.
8. **`isRunAgentSession` is exported (card 6).** Callers are taskguard.go:25 and recordupsert.go:29, both on card 6. Covered.
9. **New `runproc` role (card 6).**
   - `subjectPassSpared` (end.go:100) stays unchanged by design.
   - The end_test.go:481 table is on card 6.
   - runproc/doc.go:9, "Their roles are `RoleSync` and `RoleSlackRelay`", is on card 6 as Extends. That sentence is an enumeration the card must rewrite (rubric: Extends on a rewritten ref, which the handler owns).
10. **New package `internal/boardserve` (cards 4 and 5) and the `lymdirs` constant (card 5).**
    - **Missed:** `docs/overview.md`'s package tree (docs/overview.md:296-375), which lists `internal/boardcli/`, `internal/boardengine/`, `internal/lymdirs/`, `internal/runproc/` and the rest, one line each. No card adds an `internal/boardserve/` line, although cards 7 and 9 already edit that file. → F2
11. **New verbs `board open` and `board serve` (card 6).** `TestCLITree_EveryCommand` (clitree_test.go:103) and `TestCLIForm_EveryCommand` (clitree_test.go:320) walk every command and pin audience, Use, Short and Long. They apply to the new verbs automatically, and no per-verb table needs a row; I did not read their bodies. `TestPayloadUse_MatchesDeclaredKeys` covers payload verbs only, and neither new verb takes a payload.
12. **The `rerender` `Short` reword (card 6).** Only cli.go:406 reads it. `stepContract` (cli_test.go:329) asserts the envelope, not the `Short`. Covered.
13. **`unboundedWaitFiles` and the `PATTERN-daemon-wakeups` list (card 5).** Covered.

**Findings**

- **F1 — LOW, consistency.** Location: card 1 (`01-board-view-projection.md`) and card 3 (`03-viewer-page-asset.md`).
  - Summary: two comments that enumerate the board's render outputs and the replay's recognised files are left out of the cards that add `board-data.js` and `board.html`. The first is render.go's file header ("Those are README.md, a project-&lt;slug&gt;.md … and a design-*.md"). The second is the comment over the replay's file-name constants at replay.go:31.
  - Fix: card 1 lists `internal/boardengine/render.go`'s header for the new render output, and card 3 lists the replay.go:31 comment.
  - Evidence: internal/boardengine/render.go:1-2 and internal/boardengine/replay.go:31-32.
- **F2 — LOW, consistency.** Location: card 4 (`04-board-server-routes.md`), which creates the package.
  - Summary: `docs/overview.md`'s package tree gives every `internal/` package one line, but no card adds `internal/boardserve/`. Cards 7 and 9 edit only the VS Code passages of that file.
  - Fix: card 4 lists `docs/overview.md` (Extends) for the `internal/boardserve/` tree line.
  - Evidence: docs/overview.md:296-297 and :369-375.

Tool calls:
1. Grep for tests asserting render file sets, the `rerender` `Short` text and board verb lists, plus docs mentions. The manifest had no readers of the render file set outside boardengine.
2. Grep of replay.go's overview and Claims structure and of the file-set asserts in render_test and ownerset tests. The manifest held no replay code.
3. Read of replay.go's `Claims`, `isBoardOverview` and `CommitRuns`. Needed to judge whether newly claimed paths break the commit-run mapping.
4. Read of replay.go's `Apply` file dispatch and the boardengine `doc.go` README section. Needed to judge whether `Apply` refuses an unrecognised claimed path.
5. Read of cli_test's `stepContract` list, the `runproc` role tables, and the `docs/overview.md` tree lines. The manifest lacked per-verb and per-role tables.
6. Grep of the `docs/overview.md` package tree and the every-command CLI tests. Needed to confirm whether the overview enumerates packages and which tests walk every verb.
