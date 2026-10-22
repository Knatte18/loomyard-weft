---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/handles.go:176
    summary: An entry whose letter is undecided keeps its stored handle even when a lower-id entry holds it, so two entries can share one handle and a handle selector resolves to the first only
    fix: In assignHandles, when claim(stored, number) returns false for an undecided entry (the !decided case), clear that entry's Handle instead of keeping the duplicate; add a handles_test.go case with a decided note and an undecided note both stored as B1.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/runlock_test.go:53
    summary: The acceptance criterion that a run-held task stored with a priority still takes a status write has no test; TestRunLock only gained a held-note priority row and TestLoad only checks the drop
    fix: Add a row to TestRunLock that stores a run-held task with a priority in board.json and asserts a status write on it succeeds, so moving the drop out of migrateEntries fails the test.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardcli/cli_test.go:574
    summary: Merge's set_status selecting by handle is claimed in the summary and the decision record but no test row drives it
    fix: Add a row beside merge_with_set_status_succeeds in the CLI merge test whose set_status object uses "handle" (for example the task's T handle) and assert the status is set.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/render_test.go:220
    summary: The readmeFixture comment still lists a high-priority task, but tasks carry no priority and the high priority sits on the note dropped
    fix: Change "a high-priority task" in the readmeFixture doc comment to "a high-priority note".
  - id: F5
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/boardengine/handles_test.go:130
    summary: The brief-listing check in TestHandles_AssignedOnLoad tests only brief[0].Handle, so it passes when later entries lack a handle
    fix: Compare every entry's Handle in the brief listing against the expected handles in id order.
---

### [LOW] Undecided-letter entry can keep a duplicate handle

**Location:** internal/boardengine/handles.go:176
**Issue:** A note whose type label is no longer configured has no decidable letter and keeps its stored handle, which is then claimed with the result of `claim` ignored. If a lower-id decided entry already holds that handle, both keep it, which breaks the guarantee that a handle names one entry; `get B1` then returns the first match. The case needs a removed type label and a collision, so it is rare.

### [LOW] No test for a run-held task stored with a priority

**Location:** internal/boardengine/runlock_test.go:53
**Issue:** The decision record's acceptance criterion says a run-held task stored with a priority loads without it and still takes a status write. `TestLoad` pins the drop and `TestRunLock` pins a held note's priority change, but nothing pins the run-lock outcome the load-time drop exists for.

### [LOW] Merge set_status by handle is untested

**Location:** internal/boardcli/cli_test.go:574
**Issue:** The change description lists merge's `set_status` among the verbs that take a handle. The lookup-contract rows cover `get`-style verbs, and no merge row passes a handle.

### [LOW] Stale fixture comment names a high-priority task

**Location:** internal/boardengine/render_test.go:220
**Issue:** The comment describes the fixture as holding a high-priority task; after this change the fixture's high priority is on the note `dropped`.

### [NIT] Brief-listing assertion checks one entry

**Location:** internal/boardengine/handles_test.go:130
**Issue:** The assertion that the brief listing carries handles reads only the first element.

## Rejected

- Promote keeps the note's dates untested (`dates.go:46`): `TestEntryDates` already states that a promote revises nothing, so a new assertion would duplicate it.
- Replay keeping origin's handle for a changed entry untested: the doc line states existing behavior, since handles are not a replayed field, and the added-entry row covers the new behavior.
- `handle must be a string` refusal untested: a trivial type branch beside rows that already pin the other malformed-handle refusals.
- Focus items on decisions 1 to 5, the README rendering, `RenderOrder`, the `--text` column, the doc updates and the summary claims were checked against the diff and held; the task's own board entry is not touched by the diff.
