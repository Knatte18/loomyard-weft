---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: pattern/PATTERN-test-speed.md:42
    summary: The new sentence says lyx gate test gives go test no per-invocation value, but run.go still hands it the gate pid and the slot variables
    fix: In pattern/PATTERN-test-speed.md line 42, narrow the sentence to what the code does, for example "`lyx gate test` strips the trace id from `go test`'s environment, since Go keys a cached test result on every variable a test reads". Keep the caveat about the remaining per-invocation inputs (the enclosing-gate pid and the slot variables) out of the invariant wording.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/gatecli/prebuild_test.go
    summary: RefreshStoredMarksAnExistingEntryUsed repeats assertions TestStorePrebuilt already makes, so it breaks test economy
    fix: Delete RefreshStoredMarksAnExistingEntryUsed from internal/gatecli/prebuild_test.go. The reuse block at the end of each TestStorePrebuilt row, and the row EntryPublishedBeforeTheCallIsKept, already pin that an existing entry is refreshed to the later time and reported as usable.
---

### [LOW] Invariant sentence overstates what the gate strips

**Location:** pattern/PATTERN-test-speed.md:42
**Issue:** The sentence states a cross-cutting rule, no per-invocation value in go test's environment, that `internal/gatecli/run.go` does not keep. It still appends the enclosing-gate pid and the slot lease's variables to the environment go test reads, and the change description itself lists them as remaining cache-key inputs.

### [LOW] Redundant helper test in prebuild_test.go

**Location:** internal/gatecli/prebuild_test.go
**Issue:** `RefreshStoredMarksAnExistingEntryUsed` asserts only the existing-entry refresh that the end-of-row reuse block of `TestStorePrebuilt` and its `EntryPublishedBeforeTheCallIsKept` row already assert through the public path.

## Rejected

- Fork finding that `RefreshStoredReportsAMissingEntryAsNotRefreshed` overlaps the `EmptyStore` row: that test asserts the `(false, nil)` contract of the helper directly, which no other test pins.
- Fork finding that the whole-diff form lacks a plan-verify failure-record case: both forms assemble the command through the same `publishFailureSteps` and runCommand code, so a second case covers no new behavior.
- Fork note on the prune window where a prune reads an old modification time before a refresh and removes the entry after the re-stat: the change description and decision record name it as a bounded residual, needing an entry idle over two hours and a refresh inside the prune's read-to-remove gap.
