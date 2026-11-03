---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/dependency.go:107
    summary: met and standsFor recurse over project membership with no visited guard, so a stored membership loop overflows the stack on every board read, and no test builds a loop
    fix: In internal/boardengine/dependency.go give met and standsFor a visited set of project slugs (an unexported helper met(slug, visiting map[string]bool) with met(slug) calling it with a fresh map), and treat a slug already on the path as unmet in met and as already listed in standsFor, so both terminate on a membership loop, a project naming itself included. Keep layer.go getDepth's project branch safe the same way, since it recurses over waitsOn members and ComputeLayers' detectCycleDFS calls met before it can report a cycle. Add a row to TestDependenciesPending in dependency_test.go with two plain projects each naming the other as its project (and one naming itself), asserting Pending returns without recursing and a case in TestComputeLayers asserting it returns a result or a cycle error instead of crashing.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/render.go:596
    summary: afterCell now drops a met project dependency and keeps an unmet one by handle, and no render test would fail if it fell back to isDone
    fix: Add a project dependency to the board fixture of the README render tests in internal/boardengine/render_test.go, a task whose depends_on names an open project (its After cell holds the project's handle) and a task whose depends_on names a plain project with every landing member done (its After cell is empty), and assert both cells exactly. Extend the existing README test table rather than adding a new test function.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/siblings.go:63
    summary: the met filter in addedSiblings now also drops an added dependency on a fully done plain project, and no siblings row reaches that branch
    fix: Add a row to TestWriteSiblings in internal/boardengine/siblings_test.go whose seed holds a plain project P with one member M that is done, a task X and a task Y sharing a module label with Y waiting on P, and act setDeps("X", "P"), expecting no siblings, since P is met and is not added as a dependency.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/render.go:13
    summary: the file header still says Ready and the layers add the handles of the open entries each waits on, while the After column now lists the entries not done and judges a project by its landing members
    fix: In the file header comment of internal/boardengine/render.go, line 13, replace "the handles of the open entries each waits on" with "the handles of the entries each waits on that are not done, a project judged by its landing members", matching the afterCell doc comment and doc.go.
---

## Findings

### [MEDIUM] met and standsFor recurse over project membership without a visited guard

**Location:** `internal/boardengine/dependency.go:107`, `internal/boardengine/dependency.go:128`, `internal/boardengine/layer.go:106`
**Issue:** `met` calls itself for every landing member of a plain project, and `standsFor` calls itself for every unmet member, and neither tracks the projects it has passed.
A stored project chain that loops, such as two projects each naming the other as its project or a project naming itself, makes both recurse until the stack overflows, which Go cannot recover from.
`checkProjectField` refuses such a loop on write, but `parentbeam.go:11` records that a hand edit can store one, and the old code never walked membership, so the board could still be read.
Now `ComputeLayers` calls `met` in `detectCycleDFS` before it can return its cycle error, so every board render, Dobby read and `lym board` verb would crash on such a board.
No row in `dependency_test.go`, `layer_test.go` or `project_test.go` builds a loop, so nothing pins termination, which the focus directive asks about directly.

### [MEDIUM] The After column's project handling has no test

**Location:** `internal/boardengine/render.go:596`
**Issue:** The decision record asks that the README's After column show a project dependency like a task one.
`afterCell` now hides a dependency `Dependencies.met` judges met, and shows an unmet project by its handle.
The diff adds no render test row, and the existing render fixtures name only task dependencies, so reverting the condition to `isDone` leaves every render test passing.

### [LOW] The siblings met filter on a done plain project is untested

**Location:** `internal/boardengine/siblings.go:63`
**Issue:** The filter now drops an added dependency on a project whose every landing member is done.
The third added siblings row keeps `P` open through an unfinished member, so it never reaches the met branch, and reverting `met` to `isDone` there leaves all siblings rows passing.

### [LOW] render.go header still describes the After column by the open entries

**Location:** `internal/boardengine/render.go:13`
**Issue:** The file header says Ready and the layers add the handles of the open entries each waits on.
The diff changed the rule to the entries not done, a project judged by its landing members, in `doc.go` and in the `afterCell` doc comment, and left this line.

## Rejected

- Lens code-correctness finding 2 (a run-held beam weighs as its deepest open member in `getDepth`): the layer doc comment states a dependency on a project not done weighs as one on its deepest open landing member, and the dependent is held either way, so the placement is the specified behavior.
- Lens code-correctness finding 3 (the write-time cycle check follows done members): the replaced check also followed done tasks, so a done entry on a cycle was refused before; this is the same rule extended to members, not a new defect.
- Lens code-correctness finding 4 (membership-only suppression with a separate stored loop): it needs an already stored membership loop, the case F1 covers, and the refusal path stays correct for every writable board.
- Lens code-affected-set finding 2 (the Ready paragraph in `doc.go`): the paragraph carries the added project sentence, the generic "open dependency" wording is not wrong, and the diff states the rule once.
- Lens code-tests finding 3 (no plain-cycle test for `findCycle`): `TestValidateDependencyGraph` in `store_test.go` has the row "a cycle is detected" asserting "cycle detected", which covers it.

The diff matches the decision record's Goal: `depends_on` accepts a project, a dependency is met when its entry is done, Dobby's hold, the layers, the After column, the cycle check and the siblings notice read one `Dependencies`, and the refusal and its error are gone from `project.go` with no leftover reference.
The change description's claims match the diff.
Neither commit needs a `Pattern-Departure:` trailer, the diff touches no `PATTERN.md` or `pattern/` entry, and no declared PATTERN change exists to land.
