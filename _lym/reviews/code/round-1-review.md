---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/gateslot/runlog_test.go
    origin: lens:code-tests
    summary: No test pins that the tier run record carries failures for a failed go-json tier or omits the key for a passing one, D2's acceptance criterion.
    fix: In internal/gateslot/runlog_test.go add a round-trip row with RunRecord.Failures set that reads back equal, and an assertion that a record with no Failures encodes without a failures key and that a record lacking the key still reads; if the wiring at internal/gatecli/tierrun.go:115 can be reached through an existing runTier test fixture, also assert there that a failed go-json tier's record holds the counter's failures.
  - id: F2
    severity: LOW
    class: scope
    location: internal/gatecli/counters_test.go
    origin: lens:code-tests
    summary: No TestCounters row has two failing tests where one name prefixes the other without a slash, so the Test+"/" parent check is not pinned against a bare prefix match.
    fix: Add a go-json row to the TestCounters table in internal/gatecli/counters_test.go with TestA and TestAB both failing and no subtest, and want both in wantFailures.
  - id: F3
    severity: NIT
    class: scope
    location: internal/adoptcli/check_test.go
    origin: lens:code-tests
    summary: No TestRunCheck row covers a failed tier with an empty Failures list, which D3 says must still read sensibly.
    fix: In internal/adoptcli/check_test.go add a case with failedTier(1) and no failures asserting the tier notice is the summary line then "log " plus the log path.
---

### [MEDIUM] Run record failures key unpinned

**Location:** internal/gateslot/runlog_test.go
**Issue:** The decision record's D2 acceptance names two outcomes: a failed go-json tier's record holds the failures, and a passing tier's record has no `failures` key.
The only test touching `Failures` is the counter table; the one-line wiring `record.Failures = counter.Failures()` in `internal/gatecli/tierrun.go:115` and the `omitempty` tag on `RunRecord.Failures` have no row that would fail if either were dropped, and the adopt tests build `RunRecord` values directly.

### [LOW] Prefix-sibling row missing

**Location:** internal/gatecli/counters_test.go
**Issue:** `Failures` drops a failed test only when another failed test in the package starts with its name plus a slash.
The existing subtest row passes under a bare `HasPrefix(other, failure.Test)` too, so a regression to that match, which would hide `TestA` behind a failing `TestAB`, goes unnoticed.

### [NIT] Failed tier without failures unpinned

**Location:** internal/adoptcli/check_test.go
**Issue:** `tierNotice` handles a failed tier with no recorded failures (a build failure, a VSTest tier) by emitting the summary and the log line.
Every failed-tier row in `TestRunCheck` carries at least one failure, so that branch has no row.

## Rejected

- Output arriving after a test's pass or fail event re-inserts a `firstLines` entry that is never read or removed (lens:code-correctness): go test emits a test's output before its terminal event, and the stale entry affects no result.
- Quadratic parent filter in `Failures()` (lens:code-correctness): it runs once per tier over the failed tests only, and the lens itself rated it not worth a finding.
- Parent test's own error line is lost when a subtest also failed (lens:code-correctness): D1 specifies leaves only, so this is the decided behaviour.
- Check view carrying the names (handler): `checkView` copies `record.Tiers`, so the new `failures` key reaches the view's JSON without a change, which satisfies D3.
- `AdvanceByWaitsWhere` bound (handler): it exists before this diff and returns after its own 10 s deadline, after which `awaitRun` fails on its timeout, so D5's step has no unbounded loop and adds no real-time sleep.
- Creel production shutdown path and other creel tests (handler): the diff touches only `server_test.go` and `cli_test.go` under creel, so the Out list holds, and both commit messages match their diffs.
