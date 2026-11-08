```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. Per behaviour, the test that pins it:**

- **Origin record with the beam mark reads as a beam** (`pairIsBeam`, `buttons.go:64`): pinned by `TestReedGeometry_ButtonContext/BeamMarkReadsBeam`. Reordering the checks would not break it, which is acceptable because the result is the same.
- **No mark and no board file reads as a task, so the board-absent path does not fail boot** (`buttons.go:67`, `ReadTasks` on an absent file): pinned by `NoBeamMarkReadsTask`. The board directory exists but `board.json` is absent (`hubWithOrigin`, `parent_test.go:124`), and `Store.Load` returns an empty list when the file is missing (`store.go:206`). A board directory that is itself absent is not tested. `Load` takes the same `fileExists` path, so this is not a gap worth a row.
- **A beam-flagged project entry matching the slug reads as a beam before the mark** (`buttons.go:73`): pinned by `BoardBeamFlagReadsBeamBeforeTheMark`. It would fail if the board read were removed.
- **A project without the beam flag reads as a task** (`buttons.go:73`, the `task.Beam` operand): pinned by `BoardProjectWithoutBeamFlagReadsTask`.
- **A malformed board returns an error** (`buttons.go:69`): pinned by `UnparseableBoardIsAnError`, which writes `{` as `board.json`. The `{` fails the unmarshal in `Load` (`store.go:216`), so `ReadTasks` errors and the wrapper at `buttons.go:69` is reached.
- **A malformed origin record returns an error**: pinned by `UnparseableRecordIsAnError`, whose board is empty.
- **A matching entry that is not a project reads as a task** (`task.Kind == KindProject`, `buttons.go:73`): **none**. `boardWith` takes a `kind` argument, but no row passes a non-project kind with `beam` true. If `task.Kind == KindProject &&` were deleted, every existing row would still pass.
- **An entry for another slug is ignored** (the `task.Slug == l.WorktreeName` match, `buttons.go:72`): **none**. `boardWith` only ever writes the entry `child-task`, which is the pair's own slug. A bug that matched any beam entry, or the first entry, would pass every row. The loop's fall-through return of `false` is hit only when the board is empty.

**Findings:**

1. **LOW, design.** `internal/hubgeom/hubgeom_test.go:~165-180` (`boardWith`, rows list). No row pins the kind check in `pairIsBeam` (`buttons.go:73`). Add a row such as `BoardNonProjectWithBeamFlagReadsTask`, using `boardWith("task", true)` with `wantPlace` task.
2. **LOW, design.** `internal/hubgeom/hubgeom_test.go:~165-180` (`boardWith`, rows list). No row pins the slug match (`buttons.go:72`): the only board entry is always the pair's own. Add a row whose board holds a beam-flagged project under a different slug, expecting `ButtonPlaceTask`. `boardWith` needs a slug parameter, or the row needs a second entry.

Not a finding: the doc.go paragraph is prose and exempt. The new rows extend the existing table, so they duplicate no other test.

**Tool calls (1):**
1. Grepped for `^func hubWithOrigin` with context in `parent_test.go`; my context lacked the helper, and I needed to know whether the board directory exists and whether the board file is absent in the no-board rows.
