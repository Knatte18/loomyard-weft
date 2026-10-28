---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/replay.go:522
    summary: The new isBoardOverview and its project-file handling in the records replay have no test of their own, and the summary names a test the diff does not touch
    fix: Add a replay test in internal/fabricengine where a board commit touches project-x.md beside board.json and README.md and replays by entry with no entry change named and no conflict. Add rows asserting that project-.md, project-a/b.md and dir/project-x.md are not overviews. Correct the summary's Tests line if it keeps naming TestSetOwners/CompetingWriteLosesToOrigins as the cover.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/fabricengine/replay.go:519
    summary: The records replay change is outside the decision record's Scope, which lists only boardengine files, and is not recorded as a parent correction
    fix: Leave the code in place, because without it a board commit touching a project file falls back to a git-level replay and conflicts on board.json. Defer to the run's parent to record the correction with lym loom decision add, or to refuse it. The fixer changes nothing.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/render_test.go:TestRenderReadmeGolden
    summary: No golden row pins the After cell for a dependency that names no board entry, which D5 says shows as a code span of its slug
    fix: Add a DependsOn slug that is not on the board to a task in one golden row of TestRenderReadmeGolden and pin its code span in that task's After cell, beside a handle so the line break joins both.
  - id: F4
    severity: LOW
    class: design
    origin: handler
    location: internal/boardengine/render_test.go:TestRenderReadmeGolden
    summary: A draft task in a project file is pinned by no row, since the old projects row's Draft case was removed and project-tidy.md is not asserted
    fix: Add project-tidy.md to the files map of the projects row, or give another project a draft task, and pin its own Draft subsection after the Ready and layered tasks and not under Ready.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/replay.go:30
    summary: boardProjectPrefix repeats the project- prefix that boardengine declares as projectFilePrefix, so the two can drift apart silently
    fix: fabricengine cannot import boardengine, since boardengine imports fabricengine, so add a comment on both constants naming the other as the file prefix it must equal.
---

### [MEDIUM] Project-file replay handling is untested

**Location:** internal/fabricengine/replay.go:522
**Issue:** `isBoardOverview` decides whether a board commit is replayed by entry or falls back to a git-level replay that conflicts on `board.json`. No test in `internal/fabricengine` mentions `project-`, and no test file changed in the diff. The summary names `TestSetOwners/CompetingWriteLosesToOrigins`, which the diff leaves untouched. Its fixture may render a project file incidentally, but it asserts only the owner outcome, so a broken prefix, slug or slash condition would not necessarily fail it.

### [MEDIUM] Replay change is not in the record's scope

**Location:** internal/fabricengine/replay.go:519
**Issue:** The decision record's Scope lists `render.go`, `render_test.go` and `doc.go` in `internal/boardengine`, and the board entry says the split is rendering only. The diff also changes `replay.go` and `doc.go` in `internal/fabricengine`. The change is forced by the new file shape, and its commit message gives the reason, but nothing records it as an amendment, and the summary lists it under Code rather than as a parent correction. It is a deviation for the parent to settle, not one the fixer reverts.

### [LOW] Unknown-dependency After cell is not pinned

**Location:** internal/boardengine/render_test.go:TestRenderReadmeGolden
**Issue:** `afterCell` shows the slug as a code span for a dependency with no board entry or no handle. Every dependency in the fixtures is on the board, so a regression in that branch passes the goldens.

### [LOW] Draft task in a project file is not pinned

**Location:** internal/boardengine/render_test.go:TestRenderReadmeGolden
**Issue:** The code keeps drafts through the per-project filter, since it reads `own`. The only draft golden is root-level. The removed projects row held a draft in project `tidy`, and `project-tidy.md` is not in the `files` map, so a draft dropped from a project file would pass.

### [LOW] Project file prefix is declared twice

**Location:** internal/fabricengine/replay.go:30
**Issue:** `boardProjectPrefix` and `projectFilePrefix` in `internal/boardengine/render.go` must hold the same text, and nothing ties them together.

## Rejected

- Unreachable entries when a project names a slug not on the board (code-correctness F1): a write cannot store a dangling project, since prune keeps a done project while a member names it, so only a hand edit of `board.json` reaches it, and the code guidelines rule out handling a state the code's own guarantees prevent.
- `isBoardOverview` accepting a hand-written `project-notes.md` (code-affected-set F3): the board directory is board-owned and the lens itself calls it acceptable.
- `ProjectRemoved` removes the only entry (code-tests F3): the lens asks for no change, and the rename test covers removal beside other entries.
- `docs/overview.md` wording about the Running table (code-affected-set note): still true in substance, and not a finding.
- The focus items on D1 to D6 behavior, the header up link, `Research` heading and ordering, the After cell and the doc.go `# README` section: each settled as correct with no defect, so they produce no finding.
