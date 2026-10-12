---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/battencli/overview_test.go:355
    summary: TestOverviewSource_ReadsThePrimesRunsFromDisk fourth and fifth frames still name the third run strict, but the merged-in first part of the test names it future-key
    fix: In the fourth-frame want list (overview_test.go:346-350) and the fifth-frame want list (overview_test.go:360-364), replace the third row with the future-key row, and re-pad every row of both lists to the 10-character slug column. Fourth frame rows are "run-a       paused 10m  Run-Shed  2h00m  2h00m  run-a", "run-b       paused 4m   Plan      1h00m  1h00m  run-b" and "future-key  paused 4m   Plan      -      -      future-key". Fifth frame rows are "run-a       running  Run-Shed  2h00m  2h00m  run-a", "run-b       running  Plan      1h00m  1h00m  run-b" and "future-key  running  Plan      -      -      future-key".
  - id: F2
    severity: BLOCKING
    class: consistency
    location: cmd/lyx/testdata/test-budget.yaml:4
    summary: cmd/lyx has 80 top-level tests against a budget of 79 once shortcuts_test.go adds TestShortcuts_FileAndRootListing, so TestTestBudget_PackagesStayWithinBudget fails
    fix: In cmd/lyx/testdata/test-budget.yaml change the row cmd/lyx from 79 to 80, in the same commit as the test. Card 5 added shortcuts_test.go without listing the budget file under Edit.
---

### [BLOCKING] Overview test fixture disagrees with the merged-in slug name

**Location:** internal/battencli/overview_test.go:346-364
**Issue:** The branch's pause frames (card 1) were written when the third seeded run was named `strict`.
The parent branch's merge renamed that run to `future-key` in the same test (lines 298-326), so the fourth and fifth frames, which are the branch's own additions, no longer match the rendered output.
`.lyx/verify/verify-3.log` shows the 4-space padding difference and the `strict` against `future-key` difference.
The production code in `internal/battencli/overview.go` renders correctly; only the expectations are stale.

### [BLOCKING] cmd/lyx test budget exceeded by one test

**Location:** cmd/lyx/testdata/test-budget.yaml:4
**Issue:** `cmd/lyx/shortcuts_test.go` adds one top-level test, and the `cmd/lyx` row stayed at 79.
`TestTestBudget_PackagesStayWithinBudget` reports `cmd/lyx: 80 top-level tests, over its budget of 79`.
The branch added the `internal/shortcutcli` row but not the `cmd/lyx` one, which CLAUDE.md requires in the commit that adds the test.

### Reviewed without findings

The rest of the range was read against the plan's cards and raised nothing:

- The pause marker in `overview.go` reads the record once per poll and degrades to no marker on a read error.
- The `outstanding` change in `internal/seatengine/work.go` reads session state before the up line.
- `PasteSubmit` loads its buffer from stdin through `runStdin`.
- The `shortcutcli` routing and mount are in place, and the `cmd/lyx` tests skip shortcuts in the module checks.
- The loom status strand share is an eighth.
- Each changed doc comment and `doc.go` follows the surrounding conventions.
