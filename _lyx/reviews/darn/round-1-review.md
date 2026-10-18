---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/reportengine/weeks.go:127
    summary: Week.Recipes and the Runs by recipe column have no test with a week holding both recipes
    fix: In internal/reportengine/weeks_test.go, add a darn run to the week fixture that TestRenderMarkdown or the computeWeeks test builds, or add a row, and assert both recipe counts (for example "loom 2, darn 1") so a miscount or a dropped recipe fails.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/reportengine/timingspans.go:209
    summary: No darn row pins the Run-Shed change-producer lookup through the run's own recipe table
    fix: In internal/reportengine/timing_test.go, add a batten Run-Shed entry on the darn run of the new TestComputeTiming darn row, whose change names Darn-Write or a Darn-Burler row, and assert its time lands under write or review rather than the loom mapping.
    origin: lens:code-tests
  - id: F3
    severity: NIT
    class: consistency
    location: internal/reportengine/runs.go:1
    summary: File header still says a run is full when its loom records show a finished loom run
    fix: Reword the first line of the runs.go header so a full run is one whose records show a finished run of a selected recipe, loom or darn.
    origin: lens:code-affected-set
  - id: F4
    severity: NIT
    class: consistency
    location: internal/reportengine/timingspans.go:193
    summary: phaseOf doc comment says a loom row the phase table does not name, but the row can be a darn row
    fix: In the phaseOf doc comment, replace "a loom row the recipe's phase table does not name" with "a child row the recipe's phase table does not name".
    origin: lens:code-pattern-fit
---

Verdict: the diff makes the change the board entry asks for.
Run selection admits finalized loom and darn seeds in both reports and keeps a batten seed or an unfinalized tag partial.
Each run carries its recipe, the closed phase table is per recipe, and the medians are per recipe.
`lyx gate test ./internal/reportengine ./internal/reportcli` passes.

### [LOW] Mixed-recipe week untested

**Location:** internal/reportengine/weeks.go:127
**Issue:** `computeWeeks` fills `Week.Recipes`, and `wakeRows` renders it, but the only assertion is on a loom-only fixture. A bug that merged recipes or dropped darn from the count would pass.

### [LOW] Darn Run-Shed change lookup untested

**Location:** internal/reportengine/timingspans.go:209
**Issue:** The Run-Shed change path looks up the changed producer in the run's own recipe table, but the existing change rows use loom producers only. Looking the producer up in loom's table would pass today's tests.

### [NIT] Stale file header

**Location:** internal/reportengine/runs.go:1
**Issue:** The body comments and `doc.go` were updated for darn, the file header was not.

### [NIT] Stale "loom row" wording

**Location:** internal/reportengine/timingspans.go:193
**Issue:** `phaseOf` also classifies darn rows now.

## Rejected

- `Week.Recipes` and the Runs by recipe column as a scope excess: the entry asks for runs tagged with their recipe, and convergence markdown has no per-run table, so the weekly count is where the tag shows.
- `internal/reportcli/doc.go` not touched although the decision record lists it as in scope: the record is the writer's claim, and the doc names only envelope keys that are unchanged.
- Dropping the `Darn` family from loom's phase table: the loom recipe holds no Darn row, and a darn run uses its own table.
