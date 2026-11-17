```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that fails if it breaks.

**Each behaviour and the test that pins it**

1. **`lym-batching` and `lym-plan-edits-before-gate` now target `agents-file`.** Pinned by the `ShippedEntriesArePinned` rows (`internal/heddleengine/store_test.go:184-185`). Pinned a second time by `BothFilesHoldTheShippedLayerWithNoEntry`'s heading loop (`render_test.go:362-366`), which fails if either entry stops reaching `AGENTS.override.md`.
2. **The shipped section renders into `AGENTS.override.md`.** Pinned by:
   - `BothFilesHoldTheShippedLayerWithNoEntry`, which compares the whole file to `codexFileWant("")` and checks the written list (`render_test.go:358-369`);
   - `ClaudeFileJoinsEntriesInOrderIntoItsFileAlone` (`render_test.go:268-273`);
   - the `CodexFileOpensWithRootAgents` rows (`render_test.go:331-334`).

   `fileTargetWant` decides which entries to expect from each entry's own `Targets`, independently of production's `reachesFileTarget`. So if production dropped `agents-file` entries from `codex-file`, the expected file would still include them and the comparison would fail.
3. **The shipped section still renders into `CLAUDE.local.md`.** Pinned by every `claudeFileWant` comparison, for example `render_test.go:265` and `:355`, because `fileTargetWant` includes the `agents-file` entries.
4. **Order in `AGENTS.override.md`: the opening file, then the shipped heading and texts, then the repository heading and texts.** Pinned by the full-file comparisons in `AgentsFileRendersIntoBothFiles` (`render_test.go:291`) and `CodexFileOpensWithRootAgents` (`:331-334`).
5. **The overview lists the shipped entries under `agents-file`.** Pinned by `TestRenderOverview` (`overview_test.go:74`). If the entries went back to `claude-file`, the `agents-file` section would lose "- From: shipped with lym" and the check would fail.
6. **A stale marked `AGENTS.override.md` is removed when its target has no entry, even when the file is missing from the exclude list.** Before this change, the parallel subtest got that empty target from the real shipped layer. Now `TestRenderRemovesAStaleMarkedRenderEvenOffExcluded` uses an empty shipped layer (`render_test.go:240-249`) and fails if the file is kept or the removal goes unreported.
7. **A pull that takes in a changed `AGENTS.md` re-renders `AGENTS.override.md`, with the shipped section now inside it.** Pinned by `TestSeam_PullTakeIn`'s prefix and suffix checks (`seam_integration_test.go:94-96`): the opening text must be the new `AGENTS.md`, and the file must end with the store's entry. The exact shipped text in between is pinned by the unit tests in items 2 and 4.
8. **The reworded lines in `shipped.json`.** These are static prose and exempt.

**Findings:** none. Every behaviour above has a test that fails if it breaks. Tier 0 of `lym gate test ./internal/heddleengine` passed in the handler's run.

One note outside this lens: the heading loop in item 1 overlaps with `ShippedEntriesArePinned` plus the exact comparison. The board entry asks for that check, so whether the overlap is a test-economy finding is for the handler to decide.

Tool calls: none.
