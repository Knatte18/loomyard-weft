---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubgeom/hubgeom_test.go:165
    summary: no row in TestReedGeometry_ButtonContext pins the project-kind check in pairIsBeam
    fix: In TestReedGeometry_ButtonContext add a row BoardNonProjectWithBeamFlagReadsTask that uses boardWith("task", true) with the record {"parent_branch":"main"} and wantPlace ButtonPlaceTask, so deleting the task.Kind == boardengine.KindProject operand in internal/hubgeom/buttons.go fails a row.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubgeom/hubgeom_test.go:165
    summary: no row pins the slug match in pairIsBeam, since the only board entry is always the pair's own slug
    fix: Give boardWith a slug parameter, then add a row BoardBeamOfAnotherSlugReadsTask whose board holds a beam-flagged project under a slug other than child-task and expects ButtonPlaceTask, so a pairIsBeam that matched any beam entry fails a row.
---

### [LOW] No row pins the project-kind check

**Location:** internal/hubgeom/hubgeom_test.go:165
**Issue:** `pairIsBeam` returns true only for `task.Kind == boardengine.KindProject && task.Beam` (`internal/hubgeom/buttons.go:73`).
`boardWith` takes a kind, but no row passes a non-project kind together with the beam flag.
Deleting the kind operand leaves every existing row green.

### [LOW] No row pins the slug match

**Location:** internal/hubgeom/hubgeom_test.go:165
**Issue:** `pairIsBeam` matches the entry by `task.Slug == l.WorktreeName` (`internal/hubgeom/buttons.go:72`).
`boardWith` always writes the one entry `child-task`, the pair's own slug, so a match on the first entry or on any beam entry passes every row.
The no-match fall-through is reached only on an empty board.

## Rejected

- A board read failure turns off the buttons of every non-beam task session (lens code-correctness, F1): `ParentOf` in `internal/hubgeom/parentof.go` returns the same board read error, `composeButtonRowLocked` already degrades to no buttons with a Warn, and the row `UnparseableBoardIsAnError` pins the choice. A corrupt `board.json` fails the rest of lym too.
- `pairIsBeam` has no done check, unlike `ParentBeam` and `ideengine/spawn.go:338` (lens code-affected-set, F-A): the board slug is unique, so a done beam's slug cannot be reused by a later pair, and the done beam's origin record already carries the mark. The place cannot differ from the origin-mark answer, so there is no observable effect.

Checked for the focus questions:

- The board-absent path: `Store.Load` returns an empty list with no error when `board.json` and the legacy files are absent (`internal/boardengine/store.go:181-256`), so a hub without a board reads as task and reed boots.
- The button dir: `buttonrow.go` uses `e.geom.AnchorPath`, the session's own worktree, for any place but task, so the beam place fixes the dir along with the group.
- Batten: `buttonContext` is the only place chooser reaching reed, and the batten run changes neither it nor the board flag, so no other path produces the task group for a beam.
- The match key: `ParentOf` uses the same slug equals worktree name key.
