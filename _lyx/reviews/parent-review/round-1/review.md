---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7, and the acceptance criterion on narrower runs discovering zero
    summary: D7 lets a narrower run that discovers zero tests pass with a warning, against the board entry's operator decision that zero discovered tests is a failure
    fix: In D7 keep the board's rule for every run, whole or narrower - a tier run that executes and discovers zero tests fails. Meet the legitimately empty case before the run instead - the scoper drops every target that holds no tests (a Go package with no Test, Benchmark, Fuzz or Example functions, TestMain alone counting as none; a C# project that is not a test project), and when no target is left the tier is reported as skipped with nothing to test, a distinct outcome that is neither a pass nor a failure and is logged as such. A run with words after -- that discovers zero fails too. Keep the whole-run min_tests floor as it is. Drop the departure sentence and the Bound paragraph, and change the acceptance criterion to - a narrower run over targets that hold tests and discovers zero fails; a scope with no test-holding target is skipped and reported as nothing to test.
---

### [BLOCKING] D7 departs from the board's zero-test rule

**Location:** `_lyx/discussion/decision-record.md` D7, and the acceptance criterion "a narrower or filtered run discovering zero passes with the warning".

**Issue:** The board entry's operator decisions (2026-10-21), under "Verify runs are bounded and observable", say that zero discovered tests is a failure, with no exception for narrower runs.
The parent rules for the operator here: the rule stands.
The vacuous pass it guards against is not confined to whole runs: in NORCE Models a card-scoped run of a gated project with its MSBuild property dropped discovers nothing and exits green, and D7's Bound paragraph concedes that such a run passes with only a warning.
The case D7 rests on, a scope that legitimately holds no tests, is real but is met without the departure: a target with nothing to test is not a run that discovered zero, so the scoper leaves it out, and an empty remaining scope is a skip, reported as such.
A filter after `--` that discovers zero is most often a typo, and failing it is what the operator asked for.
