---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/render.go:4
    summary: Two render.go comments still list the Tasks subsections without Draft
    fix: In internal/boardengine/render.go, line 4 (the file header) and line 238 (the renderReadme doc comment), change "dependency layers and Independent" to "dependency layers, Independent and Draft".
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/boardengine/render.go:442
    summary: The standingCell doc comment does not name Draft among the standings it can return
    fix: In internal/boardengine/render.go, extend the standingCell doc comment to "Done once finished, the run's place while a run holds it, Ready, Note, Project, Draft, or the name of its dependency layer".
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/boardengine/doc.go:176
    summary: The Ready sentence still says Ready holds the open tasks with no open dependency, which a draft task with none no longer satisfies
    fix: In internal/boardengine/doc.go line 176 write "Ready holds the open, released tasks with no open dependency", and in internal/boardengine/layer.go line 29 write "An open task that is not a draft and has no open dependency is readyLayer".
---

### [LOW] Stale subsection lists in two render.go comments

**Location:** `internal/boardengine/render.go:4`, `internal/boardengine/render.go:238`
**Issue:** Both comments say Tasks split into "Running, Ready, dependency layers and Independent".
The diff adds Draft as a Tasks subsection and edited the neighbouring comment at `render.go:317`, but not these two, so they now disagree with `doc.go`, `layer.go:3` and the code.

### [LOW] standingCell doc comment omits the new Draft standing

**Location:** `internal/boardengine/render.go:442`
**Issue:** The comment lists Done, the run's place, Ready, Note and the dependency layer name.
A draft member now reads `Draft`, which only the fall-through to `layerSection` produces, so the comment no longer states a standing this change introduces.

### [NIT] Ready sentence no longer exact

**Location:** `internal/boardengine/doc.go:176`
**Issue:** "Ready holds the open tasks with no open dependency" is false for an open draft task with no dependency, which sits under Draft.
The earlier Draft sentence in the same paragraph states the exception, so a reader can recover it, and `layer.go:29` has the same wording.

## Rejected

- Cycle detection through a draft task has no test row (lens code-tests, LOW): the diff leaves the cycle DFS unchanged, and it walks every non-done task regardless of layer, so no new behaviour lacks coverage.
- The combined `TestComputeLayers` draft row hides which sub-case failed (lens code-tests, NIT): the row follows the table style of its neighbours and each slug's expected layer is named in the failure diff.
- A run-held draft task is uncovered (lens code-tests): the state is unreachable, since a draft entry holds no run lock, and `RunHolds` is checked before the draft test.
- `docs/overview.md:359` lists the Tasks subsections without Draft (handler): the sentence summarises the module and already omits Independent, and the task's rule asks for `overview.md` only when the module table or execution stack changes.
