---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/gateslot/runlog.go:188
    summary: FailedTier returns the last failing tier line, but a call with several failed runs now exits with the first failing run's code, so the landing failure record names a different tier than the exit code
    fix: In internal/gateslot/runlog.go change FailedTier to scan the log forward and return the first tier summary line whose exit code is not 0, matching the exit-code rule, and reword its doc comment to "first". Update the same "last failing tier" wording in internal/landingshed/verifygate.go (recordFailure comment near line 57 and the call at line 96), internal/landingshed/doc.go (the Merge-In and Finalize verify failure paragraph), internal/verifytree/doc.go (near line 34) and pattern/PATTERN-verified-tree.md (line 17). Extend the existing FailedTier test with a row holding two failing summary lines that expects the first.
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/run.go:140
    summary: A run that exits 0 while the run context is cancelled with no caught signal halts the call and runTest returns 0 with later tiers and guard runs unrun, a false green
    fix: In runTest's loop, where `if end.halt` returns `exitAfterSignal(caught, end.code)`, return a non-zero code when end.code is 0 (for example `code := end.code; if code == 0 { code = 1 }; return exitAfterSignal(caught, code)`), so only a caught signal turns a halt into 128 plus the signal number and any other halt is never green. Add a row to the existing cancelled-context or signal test only if a test can cancel the parent context without a signal; otherwise leave it uncovered.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/gatecli/run.go:143
    summary: A halt after earlier runs failed returns only the halting run's code and prints no closing line, so a refusal, guardStepRun error or busy hub in a later run drops the first failure's code and the list of failed runs
    fix: In runTest's loop, when `end.halt` is true and `failures` is non-empty, print the same closing line before returning, and keep the first failure's code for a halt that is not a refusal or a signal (a busy hub exit 75 after a failed tier should not read as no failure); build the line in one small helper used by both the halt return and the normal end. A refusal still writes its single JSON envelope to stdout and the closing line goes to stderr. Add the case to the failing-tier subtest of TestGateTest_Scenario, for example a later tier whose prebuild refuses.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/gatecli/test_integration_test.go:1344
    summary: No test fails a guard run, so the "guard tierN" label in the closing line and the first-failure rule across a listed failure and a guard failure are unpinned
    fix: Extend the guard-table row "a failing tier still runs the guards and returns its code" in internal/gatecli/test_integration_test.go so the guard run also fails (its own exit code differs from the listed tier's), and assert the exit stays the listed tier's code and the closing line names `tier0 (exit 3), guard tier0 (exit <code>)`; fold it into the existing row, not a new test function.
    origin: lens:code-tests
---

### [MEDIUM] FailedTier and its consumers disagree with the exit code when several runs fail

**Location:** internal/gateslot/runlog.go:188, internal/landingshed/verifygate.go:96
**Issue:** Before this change a call had at most one failing tier, so the last failing summary line was the failing tier. Now tier0 and tier6 can both fail, the call exits with tier0's code, and the landing failure record names tier6; a guard run after a failed listed tier also carries a `tierN` line and wins. The diff decided the exit code is the first failure's but left this reader and its docs on "last".

### [MEDIUM] A cancelled context with a passing run halts the call green

**Location:** internal/gatecli/run.go:140, internal/gatecli/run.go:411
**Issue:** `report` sets `halt` whenever `e.ctx.Err() != nil`, but the exit code leaves 0 only when a signal was caught. A run that exits 0 after the context was cancelled by anything other than a caught signal makes `runTest` return 0 with the later runs skipped. Before the change those runs were attempted and failed on the cancelled context.

### [LOW] A halt after a failed run drops the first failure's code and the closing line

**Location:** internal/gatecli/run.go:143
**Issue:** The record settles that a refusal, a busy hub and a signal end the call at once, but also that a call with failures exits with the first failed run's code. A later-run refusal, `guardStepRun` error or busy hub after a failed tier returns 1 or 75 and no closing line, and exit 75 is documented as no failure.

### [LOW] A failing guard run and its label are untested

**Location:** internal/gatecli/test_integration_test.go:1344
**Issue:** The acceptance criterion says the closing line names every failed run, listed tier and guard run alike. The only guard-after-failure row fails the listed tier and passes the guard run, so `runLabel`'s `guard ` prefix has no assertion.

## Rejected

- Halt cases (busy hub, signal) pinned only on the first run (lens:code-tests, LOW): the acceptance criterion says these stops end the call at once as the existing tests pin; the later-run interaction is folded into F3's test.
- Closing line wording "every later run still ran" when the last run is the failing one: wording only, no behaviour or reader depends on it.

The focus directive's questions are settled: `halt` is set for a refusal, an acquire failure and a cancelled context and never for a timed-out or empty-tier failure (the tier timeout context is derived inside `runTier` from `e.ctx`); the refusal, busy-hub and signal exit codes match the old behavior; `end == (runEnd{})` records a pass only for a clean run; guard runs come last in `planTierRuns`, and `guardRunOf` has no caller or stale comment left; the `cli.go`, `doc.go` and `docs/overview.md` sentences are updated and the `test` Long text got shorter, so the CLI-form byte cap holds.
