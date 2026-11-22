```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Per-behaviour list**

1. `board-data.js` places each entry in a project, a section and a waits-on list. Pinned by `TestRenderBoardData/EntriesStandInTheirProjectAndSection` (`internal/boardengine/view_test.go:144-174`).
   - Every fixture task and note has a literal row; the projects P1–P4 are rightly absent.
   - `T10` (`gtask`), under the done project P3, is now pinned for its section `Ready` as well as its project. The old test checked only its project.
   - The count check `len(want)` = 12 equals the old `len(tasks)-4`: 16 fixture entries minus 4 projects. It is not loosened.
2. `board-data.js` lists the sections in order. Pinned by the literal list in `TestRenderBoardData/HeaderAndSections` (`view_test.go:201-204`).
3. `RenderToDisk` leaves the caller's `Discussed` flag set. Pinned by the new assertion in `TestRenderBoardData/RenderToDiskClearsAMissingDecisionRecordButKeepsTheStoredRevision` (`view_test.go:261-263`). It moved there from the deleted `TestRenderToDiskLinksOnlyAStoredDecisionRecord`.
4. `RenderToDisk` drops a missing decision record from what board readers see. Pinned by the same subtest: `DecisionFile == ""` (`view_test.go:269`). The deleted README and design-doc checks of the Sketch link were layout of human-read files.
5. A file is rewritten in place. Pinned by `TestRenderToDiskRewritesAnExistingFileInPlace` (`render_test.go:330-359`), unchanged.
6. Output paths are accepted or refused. Pinned by `TestRenderToDiskOutputPaths` (`render_test.go:363-389`), unchanged.
7. `board-data.js` is written to disk and listed in the manifest. The diff removes one `os.Stat` of `BoardDataFile` from `SeededManifestRemovesOrphanAndWritesOutputs`, but the loop over `BoardDataFile` and `BoardPageFile` at `render_test.go:424-431` still asserts it. No coverage is lost.
8. Render's output keys honour a custom README name and design prefix. The deleted `TestRenderCustomOutputs` covered this. These still cover it:
   - `ReadmeRename` (`render_test.go:443-465`): `Index.md` is written and `Home.md` removed.
   - `ProposalPrefixChange` (`render_test.go:467-492`): `task-a.md` versus `proposal-a.md`.
   - `SeededManifestRemovesOrphanAndWritesOutputs`: `prop-a.md`.
   - `EntriesCarryTheirStoredValues` (`view_test.go:210`): the `design-&lt;slug&gt;.md` value that `board.html` reads.
9. A design doc is written only for an entry with a body. The deleted `TestRenderDesignDocGoldens` asserted `design-bare.md` was absent. The absence is still pinned by `prop-b.md` (`render_test.go:435-437`) and `BodyLoss` (`render_test.go:494-516`).
10. Every project, open or done, gets a `project-&lt;slug&gt;.md` file. The deleted golden's `projectFiles` row covered this.
    - An open project's file is still pinned by `ProjectRemoved` (`render_test.go:518-538`).
    - A **done** project's file has no test now. Nothing in the diff or the context shows another reader relying on it. `replay.go:108-115` `isBoardOverview` classifies paths by prefix only and does not need the file to exist. The finding below records the gap.
11. The `lym board list --text` and `find --text` listing kept these tests (`internal/boardcli/verbs_test.go`), with a sketches-last check:
    - `stepListAndFindText` (`verbs_test.go:340-398`) still pins exact lines, the root P0 row's placement, the label filter and the sort order.
    - `stepListShowsASketch` (`verbs_test.go:401-413`) pins the sketches-last rule.

    So the deleted `TestRenderCompact` leaves no behaviour another reader uses untested. Its multi-byte width, optional-column and empty-input rows were layout alone.
12. `TestFormatAge` is unchanged; `FormatAge` keeps its test.
13. The `layer_test.go:199` keep-reason only changes comment text and alters no behaviour.

**Findings**

- **NIT**, class `consistency`, `internal/boardengine/render_test.go:518` (`TestRenderToDiskManifestCleanup/ProjectRemoved`).
  - Summary: no remaining test pins that a done project still renders to its own `project-&lt;slug&gt;.md`. The deleted golden row's `projectFiles` list covered it (`project-old.md`, diff 002 line 617). The file name is a Render contract stated in the doc comment at `render.go:121`, but a person is its only reader that the context shows.
  - Fix: if the contract matters to anything but people, add a done project to the `ProjectRemoved` fixture and assert that its file is written. Otherwise there is nothing to do.

No other finding from this lens.

Tool calls: none. The inherited context already held every test and definition this lens needed.
