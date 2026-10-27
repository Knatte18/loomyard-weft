---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/reportengine/doc.go:131
    summary: The report doc says a darn run's setup holds Preflight and Drift-Baseline, but Drift-Await now counts under setup too.
    fix: In internal/reportengine/doc.go line 131, change "A darn run's setup also holds Preflight and Drift-Baseline;" to "A darn run's setup also holds Preflight, Drift-Baseline and Drift-Await;" to match darnPhases in timingspans.go.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/recipes/darn-recipe.yaml:51
    summary: The header's hand-settle note covers only a run parked between Drift-Baseline and Darn-Write, but any in-flight darn run past Drift-Baseline lacks awaited_at under the new binary.
    fix: In the darn-recipe.yaml header comment, widen the sentence to say a darn run that passed Drift-Baseline before this recipe gained Drift-Await has no awaited_at, halts at Drift-Verify with no counted baseline, and is settled by hand with lym drift waive; keep the no-transition-code decision and add no code.
    origin: lens:code-correctness
---

### [LOW] Report doc omits Drift-Await from darn's setup phase

**Location:** internal/reportengine/doc.go:131
**Issue:** `darnPhases` in `internal/reportengine/timingspans.go` maps Drift-Await to setup, and the test table pins it, but the package doc's darn sentence still lists only Preflight and Drift-Baseline. The loom sentence two lines above names both Drift rows.

### [LOW] Recipe header understates the runs a hand settle covers

**Location:** contracts/recipes/darn-recipe.yaml:51
**Issue:** The old waiting capture never set `awaited_at`, and Drift-Await is its only writer. A darn run already past Drift-Baseline, including one in Darn-Write, therefore reaches Drift-Verify under the new binary with a passed capture that `CountedCapture` refuses. The behaviour matches the operator's no-transition-code decision; only the header's wording is narrower than the affected runs.

## Rejected

- The reportengine `timingspans.go` and `timing_test.go` edits are not out of scope: without the Drift-Await mapping a darn run raises an unmapped-row finding, so they are required follow-through, even though the decision record assumed no change.
- The merged `BaselineUnknownKey` and `BaselineCaptureKey` rows are not a test-economy defect: the unknown-key refusal stays covered.
