---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/loomcli/sharedbootstrap.go:381
    summary: "The loom status window name still comes from TitleWithHandle and not the shared console title function, against the Goal's limit that the places forming a title today call it."
    fix: "Decide whether the status window of a task, named T4 plus the recipe, is a console window the Goal covers. If it is, form its name in statusWindowTitle in internal/loomcli/sharedbootstrap.go through consoletitle.Title with the hub shortname and the P# and T# handles read from the board, and update its test and the docs/overview.md line that shows T4 and tapestry. If it is not, keep the code and state in _lym/landing/summary.md that the status window is left out and why. Leave the runs window in internal/orchcli/runs.go as it is, since the Goal gives the runs strand no title of its own."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reedcli/window.go:49
    summary: "terminalWindowTitle has no test, because the windowTitle seam replaces it in every row of window_test.go, so its base-name fallback and unresolvable-prime path are uncovered."
    fix: "In internal/reedcli/window_test.go add a table-driven test that calls terminalWindowTitle through the export_test.go file of package reedcli. Give it one row for a directory in no worktree, which expects the base name, and one row for a worktree in a hub fixture, which expects the console title. Use a hub fixture at tier1 through hubforge, or an in-memory fake if one fits."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/handles.go:173
    summary: "An unreadable board titles a beam with its slug, not with its beam or orch label, so the label is lost along with the handles although the doc comments promise only the handles are left out."
    fix: "In worktreeWindowTitle in internal/boardengine/handles.go, either keep the label on a ReadTasks error, which needs the beam flag the failed read cannot give, or correct the doc comments of IDEWindowTitle and TerminalWindowTitle to say that an unreadable board titles every non-prime worktree by its slug. Add a beam row to AnUnreadableBoardLeavesOutTheHandles in handles_test.go that pins the chosen result."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/handles_test.go:221
    summary: "TestWindowTitles has no row for a task under a done beam, a task whose project is missing from the board, or an empty primeName, the branches worktreeWindowTitle adds beyond ParentBeam."
    fix: "Add three rows to the tests table in TestWindowTitles in internal/boardengine/handles_test.go. One is a task whose project is a done beam, which expects that beam's handle as its P#. One is a task whose project slug is absent from the board, which expects no P#. One calls both titles with an empty primeName on the prime's slug, which expects the slug-form title. The board constant needs a done beam and a task with a missing project added for these rows."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/commanddeps_test.go:15
    summary: "The commandDeps title test keeps its old name and comments about the slug opened with the handle, and the comments in it and in wire.go say the bare slug when no handle exists, which holds only for a hub with no shortname."
    fix: "In internal/battencli/commanddeps_test.go rename TestCommandDeps_TitleOpensTheSlugWithTheTasksHandle to a name that says Title is the task's console title, and fix its comments. Add one row that records a hub shortname and a project handle, so a call that lost either would fail. In internal/battencli/wire.go lines 980 to 982 and in the file header comment of commanddeps_test.go, replace the bare-slug wording with the left-out-part wording."
---

### [MEDIUM] Loom status window not titled through the shared function

**Location:** `internal/loomcli/sharedbootstrap.go:381`
**Issue:** The Goal's Limits say one shared function forms the title and the places that form a title today call it.
`statusWindowTitle` forms a task's status window name through `TitleWithHandle`, with no hub shortname and no `<P#>`, and the Goal's Seen section names tmux windows without the shortname as the complaint.
The summary leaves these windows out on a reading of "the `runs` strand gets no title of its own", but the status window is not the `runs` strand.
The `runs` window in `internal/orchcli/runs.go` is covered by that Goal sentence and is not part of the finding.

### [MEDIUM] `terminalWindowTitle` is untested

**Location:** `internal/reedcli/window.go:49`
**Issue:** Every row of the window test sets the `windowTitle` seam, so the real `terminalWindowTitle` never runs.
Its three behaviours, the base-name fallback, the Warn path of an unresolvable prime and the resolve-then-title path, have no test that fails if they break.

### [LOW] An unreadable board loses a beam's label

**Location:** `internal/boardengine/handles.go:173`
**Issue:** The `ReadTasks` error return comes before the beam test, so a beam on an unreadable board is titled `<shortname>: <slug>` and not `<shortname>: beam` or `<shortname>: orch`.
The doc comments say only that every handle is left out.
The existing unreadable-board row pins a task only.

### [LOW] `TestWindowTitles` misses three branches

**Location:** `internal/boardengine/handles_test.go:221`
**Issue:** No row covers a task under a done beam, where `ParentBeam` returns empty and the code falls back to `entry.Project`.
None covers a task whose project is missing from the board, or an empty `primeName`, which the doc comments of `lookFor` and `terminalWindowTitle` promise degrades to a task title.
The focus directive names the closed-beam case.

### [LOW] Stale test name and comments around the batten title

**Location:** `internal/battencli/commanddeps_test.go:15`
**Issue:** The test name and comments still describe the old "slug opened with the handle" form.
No row sets a shortname or a project, so a `commandDeps` call that lost either would not fail the test.
The comments in the test file and in `wire.go` say the title is the bare slug when the board names no handle, which holds only when the hub records no shortname.

## Rejected

- The focus's new import edge of `boardengine` into `fabricengine`: it is not new, since `board.go`, `config.go` and other files of `boardengine` already import `fabricengine`, and the `fabricengine` edge into `consoletitle` is stdlib-only and admitted by the new `PATTERN-leaf-packages` section.
- `reedcli` importing `boardengine`: no layering test or PATTERN entry forbids it, and `lym reed window` resolves its worktree through `lymcwd` as the existing placement code does.
- `TitleWithHandle` being dead: it keeps two live callers, `internal/orchcli/runs.go` and `internal/loomcli/sharedbootstrap.go`.
- The `TestLookFor` title rows duplicating `TestWindowTitles`: each `TestLookFor` row also pins colours, so the rows stay as the existing coverage of `lookFor`.
- The summary's claim that no part of lym reads a title back: the search found no reader of `window.title`, of the terminal window title or of `{title}`, so the claim holds.
