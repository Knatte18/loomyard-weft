---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedengine/run.go:278
    summary: a gated first row on an empty seed loses the resolved current_producer at the post-gate re-read, so a halt on it persists an empty current_producer
    fix: In stepLocked, inside the `if gated` block right after the status is re-read (run.go:278-289), re-apply `st.CurrentProducer = RowAt(s.Producers, st.CurrentProducer)`, and add a stepLocked test in internal/shedengine/run_routing_test.go with an empty seed, a gated first row whose gate holds, and a producer that returns Stuck, asserting the persisted CurrentProducer is the first row's name.
  - id: F2
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shedengine/run.go:85
    summary: RowAt's empty-producers branch has no test row
    fix: Add one row to a RowAt table test (or to TestProgressAt) with an empty producer list and an empty current, asserting the result is empty.
---

### [LOW] Gated first row loses the resolved current_producer

**Location:** internal/shedengine/run.go:240, 278-289; internal/shedengine/entry.go:88-97
**Issue:** `stepLocked` resolves an empty `current_producer` to the first row in memory only. For a gated row whose gate holds, `enter` calls `markEntered`, which writes only `EnteredRow`, and the `if gated` block then re-reads the status from disk, so `st.CurrentProducer` is empty again. A Stuck, failed, awaiting or blocked outcome on that row then persists `current_producer: ""` with a halted state, and `StepResult.Next` is empty too. The decision record asks that the run start at the recipe's first row for every recipe. No current recipe gates its first row (the loom recipes' gated rows are later Drift and Code-Review rows, and the batten gated rows are not seeded by `loomshed.Seed`), so the case is latent today, but the engine's new contract covers it and the code does not. Rated LOW for that reason.

### [NIT] RowAt's empty-producers branch is untested

**Location:** internal/shedengine/run.go:85
**Issue:** `RowAt` returns `current` unchanged when `producers` is empty. No test row reaches that branch, and callers guard it already.

## Rejected

- Smoke test now asserts `st.State != ""`: weak, but `internal/loomshed/seed_test.go` already pins the exact seeded status, so nothing goes uncovered.
- Removal of `TestRecipe_SeedAndResumeRowNamesExist`: it guarded only the hard-coded seed name, which no longer exists, and `TestStep_EmptyCurrentProducerStartsAtTheFirstRow` covers the recipe-independent mechanism.
- `goto` and resume with no producer name: both refuse a running status, which is all a fresh seed is, and `gotoAdmitted` already falls back to the first row.
