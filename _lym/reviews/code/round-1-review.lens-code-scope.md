```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the decision record's goal points and its `Added after Discussion` entry.

**Changes and the decisions they trace to**

1. `internal/boardengine/render_test.go` deletes the README, project-file and design-doc goldens, their helpers, `TestRenderRecipeCellTakesTheDefaultRecipe`, `TestRenderProjectFileShowsTheBodyUnderTheBrief`, `TestRenderSlugCellLinksTheDecisionRecord`, `TestRenderToDiskLinksOnlyAStoredDecisionRecord`, `TestRenderSketchSection`, `TestRenderDesignDocGoldens` and `TestRenderCustomOutputs`. Traces to goal point 1, "Delete the README layout goldens". It is not moved to testdata.
2. In `render_test.go` the kept tests are `TestRenderToDiskRewritesAnExistingFileInPlace`, `TestRenderToDiskOutputPaths`, `TestRenderToDiskManifestCleanup` and `TestFormatAge`. Two edits are forced by change 1:
   - the new file header;
   - the removed duplicate `BoardDataFile` stat. The loop at `render_test.go:424-431` still asserts that file.

   The end-of-line comment moved to its own line (`render_test.go:401`) is a convention touch inside a hunk the change already edits. Traces to goal point 2 (keep `RenderToDisk` in-place rewrite and manifest cleanup).
3. `internal/boardengine/view_test.go` drops the README heading and table parsing (`readmePlacements`, `rootSectionNames`, the regexps) and puts in literal placement and section lists (`view_test.go:146-159`, `:201`). Traces to goal point 1 ("the heading parsing in view_test.go") and goal point 2 ("board-data.js holding each entry"). It also matches the added entry's "rewrites view_test.go's placement check as literals".
4. The `view_test.go:261-263` `Discussed`-flag assertion is relocated from the deleted `TestRenderToDiskLinksOnlyAStoredDecisionRecord`. It is forced by change 1, because that test held the only check of this behaviour.
5. The `internal/boardengine/layer_test.go:199` keep-reason rewording is forced by change 1. The old reason named the deleted render goldens.
6. Deleting `internal/boardcli/text_test.go` traces to goal point 3, the sweep.

**Decisions and the changes that build them**

- Goal point 1, delete the README goldens and the `view_test.go` heading parsing, no testdata move: built by changes 1 and 3.
- Goal point 2, keep `RenderToDisk` in-place rewrite and manifest cleanup, and `board-data.js` holding each entry: built by changes 2, 3 and 4.
- Goal point 3, sweep the repository for other human-read renderers with layout goldens and delete those: only partly built by change 6 (finding 1).
- Added after Discussion, "No test seam": delete and rewrite tests only, no production code. Built: the diff touches only `_test.go` files (diff stat: 4 test files).

No change lacks a trace, and no `Pattern-Departure:` trailer is present or needed.

**Findings**

1. **MEDIUM, scope.** `internal/boardcli/verbs_test.go:347-372` (`stepListAndFindText`). Goal point 3's sweep stopped short: a layout golden of the same human-read output the diff judged golden-only is still there.
   - The commit deletes `TestRenderCompact` because it "pinned the compact listing's layout alone" and "nothing parses that listing" (`_lym/landing/summary.md:19-20`).
   - `stepListAndFindText` still asserts byte-exact, column-padded `list --text`, `find --text` and `list --text --label bug` output. Examples: `"T1  task     0m  0m  soon   Soon thing                      bug                    [active]\n"` at `verbs_test.go:347-350`, `wantFind` at `:358`, `wantLabelled` at `:366-367`.
   - No stencil reads `--text`: the repository grep matched only `lym quarry glyphs --text` and heddle's `--text-file`.
   - The summary's claim that the sweep found only `text_test.go` (`summary.md:18-21`) is therefore incomplete.
   - Fix: rewrite those assertions so they check what other code relies on without pinning column padding. That is which slugs appear and in what order, the P0 root-row position, the label filter and find's match, for example by comparing `strings.Fields` per line or the slug column. Or drop the exact-string compares the same way `TestRenderCompact` was dropped.

**Not findings**

The README checks at `internal/boardengine/replay_test.go:954-959`, `board_test.go:106` and `ownerset_integration_test.go:280` are substring checks. They confirm that a store write or replay re-renders the README with the recorded prime owner, which is re-render behaviour rather than a layout golden.

**Tool calls**
1. grep of `_test.go` files for README layout literals ("Prime owner:", "| # |", "# Board"), to sweep for goldens outside the diff's four files. My context held no repository-wide test sweep.
2. Read of `replay_test.go:935-962`, `board_test.go:90-110` and `ownerset_integration_test.go:268-284`, to judge whether the grep hits are layout goldens or re-render checks.
