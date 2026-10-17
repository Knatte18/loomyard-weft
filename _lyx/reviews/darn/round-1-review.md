---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/sheddriver/drivehandback_test.go:41
    summary: the new TestDriveNotices row watched blocked stop sends nothing duplicates the row watched awaiting without a notice sends nothing
    fix: Delete the row named watched blocked stop sends nothing from the tests table in TestDriveNotices in internal/sheddriver/drivehandback_test.go. The behavior is pinned by the row watched awaiting without a notice sends nothing in the same table, and end to end by the TestGoDriver_Settle row watched verify halt parks with no goto and no notice in drive_test.go.
---

### [LOW] Redundant TestDriveNotices row

**Location:** internal/sheddriver/drivehandback_test.go:41
**Issue:** The row `watched blocked stop sends nothing` replaced the removed `WatchedVerifyLine` row. It sets `watched: true` with no Relay, Question or DoneFrictionFailed and wants no notices, the same input and outcome as `watched awaiting without a notice sends nothing` at line 37. `driveNotices` reads the stop state only through its `done` check, so `blocked` and `awaiting` take the same path and the row exercises no branch the sibling misses. `PATTERN-test-economy` makes a redundant test a finding, and the covering test is named above.

## Rejected

- Fork lenses code-correctness and code-affected-set raised no findings; their per-branch runs and the repo-wide search for `WatchedVerifyLine` and the watched-notice wording found nothing left stale.
- The code-pattern-fit lens raised the same duplicate row as a `consistency` finding; merged into F1.

## Holistic check

The diff makes the change the amended board entry asks for and nothing more.
A watched verify halt still decides `driveActionHandBack`, writes a settle record and stop report and parks, and sends no notice.
The unwatched halt keeps its one goto and its run-id hand-back line.
The relayed parent notice, the question and the friction-failed done are still sent for a watched run.
`internal/loomcli/doc.go` and the `driveNotices` doc comment agree with the new behavior, and the claims in `_lyx/landing/summary.md` match the diff.
