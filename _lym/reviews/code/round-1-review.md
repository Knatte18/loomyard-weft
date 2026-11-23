---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/board.html:589
    summary: The viewer's project dropdown in the create and edit forms now offers every done project, and checkProjectField refuses each such write with "is done", so the page offers choices that always fail.
    fix: "Add a `Done bool` field with JSON key `done` to `ViewProject` in internal/boardengine/view.go and set it to true in the done-project loop of `viewProjects` (the `add` closure takes it as a parameter; false from `walk`). In internal/boardengine/board.html `projectSelect`, skip a project whose `done` is true when building the options from `state.view.projects.slice(1)`. Add `Done: true` to the P3 row expected in `TestRenderBoardData/ProjectsListTheRootThenEachOpenThenEachDoneProjectWithItsParent`."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/view_test.go:40
    summary: No test pins a done project's parent when it is not the root; the fixture's only done project P3 has no project, so hard-coding the root handle at view.go:192 passes every test.
    fix: "In `viewFixture` add a done project nested in done project gamma, e.g. `{ID: 18, Handle: \"P5\", Slug: \"epsilon\", Title: \"Epsilon\", Kind: project, Labels: []string{\"bug\"}, Project: \"gamma\", Status: stringPtr(boardengine.StatusDone)}`, and in `ProjectsListTheRootThenEachOpenThenEachDoneProjectWithItsParent` expect its row after P3 with `Parent: \"P3\"`. Update the fixture's doc comment, which counts its projects."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: internal/boardcli/text.go:106
    summary: The Goal names `lym board list --text` as where the done beam task showed under P0, and the diff leaves that listing unchanged; the writer narrowed the task to the viewer without a recorded parent decision.
    fix: "Leave the code in place and defer this finding to the run's parent. Its only sound change is the parent's own: record with `lym loom decision add` either that the task covers the board.html viewer alone, or that the `--text` listing must also keep a done task with its project, which a later round then implements in internal/boardcli/text.go."
---

### [MEDIUM] Project dropdown offers done projects that every write refuses

**Location:** `internal/boardengine/board.html:589`, fed by `internal/boardengine/view.go:190-194`.
**Issue:** `projectSelect` builds the project options for both the create form (`board.html:731`) and the edit form (`board.html:642`) from `state.view.projects.slice(1)`.
Before the diff that list held only open projects.
It now holds every done project too, and `checkProjectField` (`internal/boardengine/project.go:97-98`) refuses an entry whose project is done.
A `ViewProject` carries nothing that marks it done, so the page cannot filter these options as it stands.
The code-affected-set lens and my own review found the same defect.

### [MEDIUM] A done project's non-root parent is untested

**Location:** `internal/boardengine/view_test.go:40` (`viewFixture`), covering `internal/boardengine/view.go:192`.
**Issue:** The doc comment at `view.go:167-168` says a done project carries the handle of the project whose Done section lists it.
The fixture's only done project, P3, has no `Project`, so its expected `Parent: "P0"` equals `projectHandle`'s root fallback.
A mutant that replaces `b.projectHandle(twl.Task)` with `rootProject.Handle` survives.
The nested-beam case, a done beam inside a done beam, and the README order among done projects are both unpinned.

### [MEDIUM] `list --text`, the surface the Goal names, is left unchanged

**Location:** `internal/boardcli/text.go:106` (`withRootProjectRow`).
**Issue:** The Goal's "Seen" line names `lym board list --text`.
In that listing, `withRootProjectRow` puts the P0 row before the first project, note or done entry, and `RenderOrder` sorts every done entry last, so a landed beam task follows the P0 row.
On a board with no open project or note, it sits directly under P0.
`_lym/landing/summary.md` claims the listing is flat and leaves it as it is, on the writer's own reading that the entry's title, "Board view", points at the viewer.
That reading is plausible, but the decision record carries no `Added after Discussion` entry that narrows the Goal.
The narrowing is the parent's to confirm or refuse.

## Rejected

- Double listing of a project between the open walk and the done loop: `childProjects` keeps only open projects (`render.go:721`) and the loop keeps only done ones, so the sets are disjoint.
- The order of done entries appended after the open walk breaking `buildTree`: `buildTree` creates every node before it links any parent (`board.html:406-411`), so order is irrelevant.
- The `add` refactor changing open projects' fields: every field reads the same `Task` as before, and the P1, P2 and P4 rows pin them.
- T11's effect on other `viewFixture` tests: `TestViewerPage` asserts only the page bytes, and `TestRenderBoardData`'s entry map gains T11's row.
- Merge danger: the `two-way` call holds. `board-data.js` and `board.html` are written together by one render, an older page reading the wider list still builds its tree, and nothing is deleted or pushed.
- The assistant reviewer gave no report: its end marker reads `not started`, so this review was consolidated without it.
