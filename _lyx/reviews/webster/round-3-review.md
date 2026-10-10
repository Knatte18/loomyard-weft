---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: cmd/lyx/testdata/test-budget.yaml:4
    summary: TestTestBudget_PackagesStayWithinBudget fails because cmd/lyx holds 74 top-level tests against a budget row of 73 after the merge-in
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/refscanner.go:204
    summary: IsReadOnlyCommand doc says a quoted glob character is text wherever a positional is admitted, but a lyx segment rejects a quoted glob in the fabric diff positional
  - id: F3
    severity: LOW
    class: consistency
    location: internal/websterengine/recoverbatch.go:42
    summary: ErrRecoveryExhausted doc names only RecoverSpawnOrAttach as its source, but begin-batch's existing-report refusal now wraps it too
---

### [BLOCKING] cmd/lyx exceeds its test budget row after the merge-in

**Location:** `cmd/lyx/testdata/test-budget.yaml:4`
**Issue:** The Publish failure `TestTestBudget_PackagesStayWithinBudget` reports `cmd/lyx: 74 top-level tests, over its budget of 73`.
Card 4 (`4b1d3748d`) added `TestOwnJSONFlags` and raised the row from 72 to 73, as the card requires.
The parent branch independently added its own top-level test in `cmd/lyx` and raised the same row from 72 to 73 (`90e02e739^2` and `90e02e739^1` both hold `cmd/lyx: 73`).
The two edits touched the same line identically, so the merge (`90e02e739`) resolved without a conflict and kept 73, while the package now holds both tests.
The branch's own range is otherwise clean of this: the card did what it should, and the failure is a merge artifact the merge-in brought to light.
**Fix:** Raise the `cmd/lyx` row to 74 in `cmd/lyx/testdata/test-budget.yaml`, then re-run `lyx gate test ./cmd/lyx` to confirm the budget test passes.

### [LOW] IsReadOnlyCommand doc contradicts the lyx glob rule

**Location:** `internal/fabricengine/refscanner.go:204`
**Issue:** The doc says "a quoted glob character is text wherever a positional is admitted".
`readOnlyLyxSegment` rejects a glob character in any `lyx` word, quoted or not, and its own doc says so; `TestIsReadOnlyCommand` pins `lyx fabric diff '*'` as `false`.
`lyx fabric diff <rev>` admits a positional, so the first sentence is false for `lyx` segments.
**Fix:** Scope the sentence to `git`, for example "a quoted glob character is text wherever a `git` positional is admitted, while a `lyx` word holding one is rejected quoted or not".

### [LOW] ErrRecoveryExhausted doc is stale

**Location:** `internal/websterengine/recoverbatch.go:42`
**Issue:** The comment says the sentinel is what `RecoverSpawnOrAttach`'s refusal unwraps to.
Card 7 makes begin-batch's existing-report refusal (`internal/websterengine/beginbatch.go:463`) wrap it as well, and `internal/webstercli/beginbatch.go` maps it to `recovery_exhausted`.
The comment names one producer of a shared sentinel and is now incomplete (`code-quality`: no enumerated consumer lists).
**Fix:** State what the sentinel means, a batch at the recovery cap refused, without naming the producing function.
