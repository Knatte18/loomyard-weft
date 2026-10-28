```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change the diff makes, I list the set it forces outside the touched lines.

**Per change: set found, covered or not**

1. **`renderReadme` and `renderProjects` replaced by `boardView.renderBoardFile`; `standingCell`, `projectMembers`, `codeList` and the Member, Kind and Standing columns removed.**
   - Set: the whole repo. No reference to a removed symbol survives. The only `codeList` left is the unrelated `cmd/testtiming/redundancy.go:285`, and the only `renderReadme` hit is a research data file.
   - Covered.

2. **The render output now has the `project-<slug>.md` file shape.**
   - Readers of that shape: `RenderToDisk` and its manifest cleanup (`render.go:39-87`), `ownerset.go:242`, `board.go:157` and `boardengine/replay.go:255`. All go through `RenderToDisk`, so the manifest covers the new files. Covered.
   - `fabricengine/replay.go`: `isBoardOverview` is used in `isBoardPath` and in the `localBoardEntries` filter at line 637. Line 755 uses `isBoardPath`, which now accepts project files. Covered.
   - `fabricengine/doc.go:1205`: updated. Covered.
   - `boardengine/doc.go` `# README`: updated. Covered.
   - `docs/overview.md:582` says "the board README shows where each run stands". The Running table now sits in the project file for a project member, so the sentence is loose. It is not wrong enough to raise.
   - No `PATTERN.md` or `pattern/` entry describes the board README layout, and none was declared as changed. Covered.

3. **After cell changed from slugs to handles.**
   - Readers: only the `waitTable` closure, and the goldens in `render_test.go` were updated.
   - The design docs keep their `Depends on:` by slug, and `renderDesigns` is untouched. This matches the summary's claim.
   - Covered.

4. **Notes order and the `Research` heading.**
   - Readers: `typeHeading` and `notesTypeOrder` are called only from `renderBoardFile`. The golden has a research type. Covered.

5. **Rename test (`rename_test.go`).**
   - Updated to read `project-grouping-two.md` and to assert that `project-grouping.md` is gone. Licensed, because `new` is now rendered in its project's file.

**Findings**

- **F1, LOW, consistency.** `internal/boardengine/render_test.go` and the summary claim fabric replay coverage that does not exist.
  - The summary (`_lym/landing/summary.md`, Tests) says `TestSetOwners/CompetingWriteLosesToOrigins` "covers the replay of a board commit that touches project files". That test is at `internal/boardengine/ownerset_integration_test.go:448`, and the diff touches no test there.
  - No test file in `internal/fabricengine` mentions `project-`. The `isBoardOverview` change in `internal/fabricengine/replay.go:522-554` therefore has no test in the diff.
  - Fix: add a row to the fabricengine replay tests where a board commit touches `project-x.md`, `README.md` and `board.json` and replays by entry without a conflict. Alternatively, drop the summary claim. This is a test gap for the next lens as well.

- **F2, LOW, design.** `isBoardOverview` in `internal/fabricengine/replay.go:547-554` is a second place that knows the board's `project-` prefix.
  - The prefix is a constant in `boardengine/render.go` (`projectFilePrefix`), and fabricengine redefines it as `boardProjectPrefix`. The two can drift silently.
  - `boardengine` already exports `Outputs` and similar, so the fabric side could take the prefix from there. Alternatively, a comment on both constants can name the coupling.
  - Fix: export the prefix from `boardengine` and use it in `replay.go`, or add the comment.

- **F3, LOW, design.** `isBoardOverview` accepts any `project-*.md`, including a path like `project-.md`, which is rejected only by the empty slug check, and a hand-written `project-notes.md` that is not a rendered file. The board directory is board-owned, so this is acceptable. Raised only because the filter now treats such a file as "names no entry change". No fix is needed beyond awareness. Treat as NIT if the handler prefers.

**Tool calls**
1. grep for removed symbols, `Outputs`/`.Readme` consumers and board-path consumers: to find every consumer of what the diff changed.
2. grep for staging and layout mentions in docs and `PATTERN.md`: to check for descriptions the diff left stale.
3. grep for `RenderToDisk` and manifest callers: to confirm every render path uses the manifest.
4. grep for `CompetingWriteLosesToOrigins` and `project-` in tests: to check the summary's test claim.
