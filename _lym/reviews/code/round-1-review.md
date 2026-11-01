---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/beamcli/dobbydeps_integration_test.go:75
    summary: The every-seam-filled check for both dobbies' RunNotices omits the new NewestAgentEnd seam, so a dobby built without it passes.
    fix: In the condition at line 75, add `|| notices.NewestAgentEnd == nil` after `notices.MarkerPath == nil`, so the existing "want every seam filled" error covers the new seam.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/runnotice_test.go (TestRunNotice_Conditions table), code at internal/dobbyengine/runnotice.go:269
    summary: No row pins that a nil NewestAgentEnd seam behaves as before, nor that the seam is left unread when Agents reads a run.
    fix: Add a row to the TestRunNotice_Conditions table that sets the harness seam's endErr and a read run in the fake and expects the run path's outcome (no error-suppressed notice). Add a second row that sets the harness deps' NewestAgentEnd to nil with no run read and expects the pre-change fallback from since and history.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battencli/wire.go:673 (readNewestAgentEnd)
    summary: Only the absent-worktree error of the NewestAgentEnd seam is tested; nothing shows the success path reads the ended runs of the row's own task worktree.
    fix: In internal/battencli, add a test next to TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree that writes an ended run.json under the task worktree's run directory and another under a sibling worktree's, calls DobbyRunNotices(...).NewestAgentEnd for the task slug, and asserts it returns only the task worktree record's modification time.
---

### [LOW] Every-seam-filled check omits NewestAgentEnd

**Location:** `internal/beamcli/dobbydeps_integration_test.go:75`
**Issue:** The test asserts both dobbies' `RunNotices` have every seam filled but lists six seams and not the new one. Dropping the wire in `DobbyRunNotices` would leave `NewestAgentEnd` nil, and `noticeCondition` treats nil as "no end read", so the regression this task fixes would return silently.

### [LOW] Nil seam and seam-unread-with-runs are untested

**Location:** `internal/dobbyengine/runnotice_test.go`, `internal/dobbyengine/runnotice.go:269`
**Issue:** The harness always sets `NewestAgentEnd`, so the `seams.NewestAgentEnd != nil` guard and the `len(runs) == 0` guard have no row that fails if removed: a nil-seam dereference or a seam error suppressing the notice of a row that has runs would pass every test.

### [LOW] Success path of readNewestAgentEnd is untested

**Location:** `internal/battencli/wire.go:673`
**Issue:** The focus question of whether the end time is scoped to the task worktree of the row judged is answered by the code (the closure resolves the slug's task worktree and passes its `AnchorPath()`), but no test pins it; only the absent-worktree refusal is asserted.

## Rejected

- Tests lens, history newer than end has no row: the loop's treatment of `NewestHistory` is unchanged in kind and existing history-only rows fail if `NewestHistory` is dropped from the loop.
- Tests lens NIT, stat-race skip in `ReadNewestRunEnd` uncovered: a vanished directory is a benign race not reachable without a seam, and the skip has no observable effect beyond the zero time.
- Tests lens, the failure-episode recover path of the new seam is untested: a trivial log-only path.
- Correctness lens reported none; the handler's own check of the ended-outcome coverage (done, died, stopped, an absent or unreadable record, an absent root) and of the `running` path left unchanged found none either.
