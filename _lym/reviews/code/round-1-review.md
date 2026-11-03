---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/driftengine/await_test.go:84
    summary: Neither half of emptyUnderSkips is pinned by a negative row, so a capture that failed for another reason could take the rescope Stuck with no test failing
    fix: In TestDriftAwait in internal/driftengine/await_test.go, add a row with a failed record that has Skip set and WroteNoFile false (for example Reason "exit 3"), with a scope record of partial; expect Stuck with wantReason holding capturePath and deferPath and wantNoWays holding "--scope". Add a second row with WroteNoFile true and Skip empty that expects the same plain capturePath way forward.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/driftengine/capture_test.go:110
    summary: The singular wording branch of noFileReason, one skipped test, is covered by no test, since baseline_test.go writes its own reason string and the only RunCapture skip row uses two tests
    fix: In TestRunCapture in internal/driftengine/capture_test.go, add a row after the two-test skip row with command "true", skip []string{"TestA"}, wantState CaptureFailed, wantReason "exit 0 but wrote no file to ", wantWroteNoFile true and wantNamed []string{"scope partial skipped 1 test,"}.
    origin: lens:code-tests
  - id: F3
    severity: NIT
    class: design
    location: internal/driftengine/capture.go:150
    summary: noFileReason counts len(skip) of the raw scope list, which RecordScope does not deduplicate, so a repeated --skip test is counted twice while the compare's SkipList counts it once
    fix: In noFileReason in internal/driftengine/capture.go, count the distinct tests, for example by taking len(slices.Compact(slices.Sorted(slices.Values(skip)))) in place of len(skip) for both the singular check and the printed count, and import slices.
    origin: lens:code-correctness
---

### [LOW] Negative cases of emptyUnderSkips are untested

**Location:** internal/driftengine/await_test.go:84, internal/driftengine/baseline_test.go:405, internal/driftengine/capture.go:154
**Issue:** `emptyUnderSkips` is the single gate that moves a Stuck from the plain capture way forward to the scope-change ways, and the Goal does not ask for a change to the plain way of a capture that failed for another reason.
Every new Await and Baseline row sets both `WroteNoFile` and `Skip`, so dropping either condition leaves every test green.
The case of a pre-change record, which has skips and no `wrote_no_file` key, is the same gap.

### [LOW] Singular skip wording of noFileReason is uncovered

**Location:** internal/driftengine/capture_test.go:110, internal/driftengine/capture.go:146
**Issue:** The `len(skip) == 1` branch that renders "1 test" has no row through `RunCapture`.
`baseline_test.go:409` hand-writes "skipped 1 test" into a fixture record, so it never runs `noFileReason`.

### [NIT] Reason count includes duplicate skip entries

**Location:** internal/driftengine/capture.go:150
**Issue:** `RecordScope` stores `--skip` as given, so a repeated test makes the reason say "skipped 3 tests" for two distinct tests, while `SkipList` deduplicates for the compare.
The Goal asks for the count of skipped tests.

## Rejected

- doc.go:105 reads "the Capture section names" as a cross-reference against the self-contained comment rule (lens:code-affected-set, NIT): the reference is to a section of the same package doc, not to another named symbol, so the rule does not apply.

## Judged against the Goal

- The reason names scope partial, the count and that every test may have been skipped (`capture.go:141`).
- Both Stucks name `lym drift scope --scope full` or a shorter `--skip` list before `lym drift capture` and `lym loom resume`, `lym drift scope --scope off`, and `lym drift defer`.
- `rescopeWaysForward(true)` in `captureAgain` is sound, since `RunCapture` returns the record only after `Check` passed on the branch point.
- Drift-Await offers no capture once HEAD moved.
- A pre-change record reads back with `wrote_no_file` false and keeps the plain way forward.
- The diff changes nothing the entry did not ask for.
