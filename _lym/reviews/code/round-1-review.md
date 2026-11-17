---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/boardcli/cli.go:961
    summary: list --text --label leaving out the P0 row is a behavior the diff adds with no test, so dropping the len(listLabels) == 0 guard passes the suite
    fix: In stepListAndFindText in internal/boardcli/verbs_test.go, after the existing wantList check, run runCLI(t, "list", "--text", "--label", "bug") and assert the exact stdout, the prime owner line followed by the soon task line alone with no P0 row. Add no new test function.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/boardcli/text.go:91
    summary: withRootProjectRow ignores done status, so with no open project or note the P0 row lands after or between done entries, against README order and its own doc comment
    fix: In withRootProjectRow in internal/boardcli/text.go, make the IndexFunc predicate also match a done entry (t.Layer == the done layer, exported from boardengine if needed, or t.Status pointing at done), so the row goes before the first project, note or done entry, and at the end only when there is none. Reword its doc comment to match. In stepListAndFindText, extend the fixture or add an assertion with a done task and no open note so the row's place ahead of the done entries is pinned.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/boardcli/cli.go:951
    summary: Under --sort age or --sort revised the P0 row is inserted before whichever project or note sorts first by date, a place with no meaning in a date order where its missing dates put it last
    fix: In the list command's RunE in internal/boardcli/cli.go, when listSort is set, append boardengine.RootProjectBrief() at the end of tasks instead of calling withRootProjectRow, matching sortByDate's undated-last rule; state this in the list help text beside the P0 row sentence.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: consistency
    location: internal/boardcli/verbs_test.go:582
    summary: The stepProjects doc comment still says parent_beam is empty under a grouping project, while its changed assertions expect boardengine.RootSubject
    fix: In internal/boardcli/verbs_test.go, change the stepProjects doc comment line at 582 to say parent_beam is root under a grouping project and names the beam once the project is hand-marked a beam.
    origin: lens:code-affected-set
---

### [MEDIUM] The --label branch of the P0 row has no test

**Location:** internal/boardcli/cli.go:961

**Issue:** The diff adds `if len(listLabels) == 0` so that a label-filtered `list --text` leaves out the root row.
The suite's only `--label` runs are JSON (`verbs_test.go:314`, `verbs_test.go:321`), and `stepListAndFindText` runs `list --text` without a label.
Removing the guard would print an unlabelled P0 row in a filtered listing and no test would fail.
The rubric's test economy treats a behavior the diff adds with no coverage as a finding.
The assistant's report raised the same gap and is merged here.

### [LOW] The P0 row's place ignores done entries

**Location:** internal/boardcli/text.go:91

**Issue:** `RenderOrder` puts every done entry after every open entry, whatever its kind (`internal/boardengine/layer.go:192-223`).
`withRootProjectRow` inserts before the first entry whose kind is project or note, so done status plays no part.
A board of open and done tasks puts P0 last, after the done tasks.
A board of open tasks, a done task and a done project puts it between the two done entries.
In both cases the row is not among the projects D8 places it with, and the doc comment's "in README order it leads the projects" does not hold.
Only the before-the-first-note case is pinned (`verbs_test.go:347-350`).
The code-tests lens's placement-coverage finding and the empty-or-task-only part of the assistant's coverage finding are merged here.

### [LOW] The P0 row's place under --sort is arbitrary

**Location:** internal/boardcli/cli.go:951

**Issue:** `sortByDate` runs before `withRootProjectRow`, so in a date-sorted listing P0 goes in front of whichever project or note has the earliest date.
That place carries no meaning.
The row has no dates, so the sort's own rule (`cli.go:1186-1191`) would put it last.

### [LOW] The stepProjects doc comment keeps the old printed value

**Location:** internal/boardcli/verbs_test.go:582

**Issue:** The comment says `parent_beam` "is empty under a grouping project".
The changed assertions at `verbs_test.go:600-605` expect `boardengine.RootSubject`.
The code-affected-set lens, the code-tests lens and the assistant all raised this.

The diff otherwise does what D8 and the parent's "Only the printed parent_beam names the prime root" entry ask.
`ListTasksBrief` and `outputGetTask` print `root` for the prime.
A failing seam still leaves the cell empty, never `root`.
`ParentBeam`, `Board.ParentBeamOf` and `hubgeom.ParentOf` are unchanged and keep the empty slug, and `ParentBeamOf` now has an assertion that pins it.
The JSON `tasks` array returns before the row is inserted.
The P0 row takes its title from `rootProject`, a fixed value, so the focus question about a root with no title cannot arise.
The orch-role contract, the board skill, the help text and both package docs read `root` as the prime and address its orch as `<shortname>:orch`.
No stencil or spec reads `parent_beam`.

## Rejected

- code-scope F1 (`RootProjectBrief` breaks "Exports: nothing new"): the Goal's Exports line describes what this part hands to later parts. A Go export used only inside this part's own printing path is not one of those. The alternative, repeating the root title literal in boardcli, would break PATTERN-code-conventions' one-named-constant rule.
- code-scope F2 (the `--label` gate is unlicensed scope): `--label` is documented to keep only entries that carry the label, and the root carries none. Leaving the row out keeps the filter's existing contract rather than adding scope, and the help text and package doc state it.
