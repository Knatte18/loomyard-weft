---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redundantmutants.go:287
    summary: A mutant the candidate's own run left unjudged never runs under the covering tests, so the candidate is unconfirmed even when a covering test kills that mutant and nothing rests on the unjudged run.
    fix: In runMutants, queue the covering runs for each mutant a candidate's own run left unjudged (timedOut or capped for that candidate), as the loop at redundantmutants.go:287 does for each mutant in kills, with the same passes, covers, queued and ran checks. In JudgeMutants (internal/testcover/mutantverdict.go:82-85), when the candidate's own run of id is unjudged, add id to Unjudged only when no covering test killed it (kills[id][name] false for every name in c.Covering); when a covering test killed it, leave id out of Unjudged, Killed and Unique. Add a TestJudgeMutants row where the candidate's own run of a mutant is unjudged and a covering test killed it, so the candidate stays redundant. Update the JudgeMutants, MutantVerdict.Unjudged and runMutants doc comments, and the TestGateRedundant run count and its comment if the loop fixture's covering runs now start.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/doc.go:180
    summary: The package doc still says a mutant run past the test bound or the memory cap counts as a kill in lym gate redundant, which the diff reverses.
    fix: Replace internal/gateslot/doc.go:180 with a line saying that in lym gate redundant a mutant run past the test bound or the memory cap is unjudged, never a kill.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/template.yaml:6
    summary: The operator-facing comments of redundancy_test_timeout_sec and redundancy_test_memory_mb still say a mutant run in lym gate redundant counts as a kill.
    fix: In internal/gateslot/template.yaml, change the end of the redundancy_test_timeout_sec comment from "which then counts as a kill" to "which is then unjudged, never a kill", and in the redundancy_test_memory_mb comment change "a mutant run in lym gate redundant counts as a kill" to "a mutant run in lym gate redundant is unjudged, never a kill". Regenerate or update any hub-config fixture or test that pins the template's text.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundantmutants.go:298
    summary: The new requeue of a covering test that timed out on a mutant as a candidate beside other runs, and the clearing of that stop before its rerun, are reached by no test.
    fix: Add a row to TestRunMutantsGuardKilledRunIsNoKill's file (internal/gatecli/redundantmutants_guard_test.go) or a sibling test there, using its stand-in binary, in which candidate A kills a mutant and candidate B, covering A, times out on that mutant in its candidate run; assert that runMutants reruns B with that mutant and that the returned kills or unjudged hold B's rerun result, not the candidate-run timeout.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testcover/runner.go:250
    summary: No test pins testOutput stamping a line split across writes, nor ignoring an indented subtest result line before the top-level result line.
    fix: Add a stand-in test in internal/testcover/testmain_test.go that writes its RUN line, an indented subtest --- PASS line, and its top-level result line split mid-line across separate writes with an explicit flush between them, and a TestRunAlone row over it with the stepping clock asserting the Elapsed that stamps only the RUN line and the completed top-level result line.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundant.go:234
    summary: The empty-contained answer's new unconfirmed key is asserted by no test.
    fix: In the TestGateRedundant case or refusal-free fixture that reaches the empty-contained return (a package whose map holds no contained test), assert that the answer carries unconfirmed as an empty list beside candidates and refuted; if no such case exists, add the assertion to the existing test that reaches redundant.go:233.
---

### [MEDIUM] An own unjudged run makes a candidate unconfirmed even when a covering test kills the mutant

**Location:** `internal/gatecli/redundantmutants.go:287`, with `internal/testcover/mutantverdict.go:82-85`

**Issue:** The covering runs are queued only for mutants in `kills`, and `JudgeMutants` puts every mutant the candidate's own run left unjudged into `Unjudged` without consulting the covering tests.
When a covering test kills that mutant, deleting the candidate loses no check of it, so the confirmation does not rest on the unjudged run; the entry asks to list as unconfirmed only a candidate whose confirmation rests on one.
Before the change those runs counted as kills and the covering tests ran them; now a candidate the covering tests' kills would confirm is listed as unconfirmed instead.

### [MEDIUM] gateslot package doc still calls a timeout or capped mutant run a kill

**Location:** `internal/gateslot/doc.go:180`

**Issue:** "A mutant run past the test bound or the memory cap counts as a kill in `lym gate redundant`." contradicts the `record` switch at `internal/gatecli/redundantmutants.go:252-262`, which now keeps both apart from the kills.

### [MEDIUM] Gate template comments still call a timeout or capped mutant run a kill

**Location:** `internal/gateslot/template.yaml:6`

**Issue:** The `redundancy_test_timeout_sec` comment ends "which then counts as a kill" and the `redundancy_test_memory_mb` comment says "a mutant run in lym gate redundant counts as a kill".
This is the text an operator reads in the hub's gate config, and it states the behaviour the diff removes.

### [MEDIUM] The timed-out covering-candidate rerun has no test

**Location:** `internal/gatecli/redundantmutants.go:298`

**Issue:** The `!timedOut[run]` clause and the `delete(timedOut, run)` before the rerun are new behaviour.
In the `TestGateRedundant` fixture no candidate kills the loop mutants, so no covering run of them is queued (`redundant_integration_test.go:300-302`), and `TestRunMutantsGuardKilledRunIsNoKill` discards the unjudged result.
Reverting the clause breaks no test.

### [MEDIUM] testOutput's line assembly and subtest filtering have no test

**Location:** `internal/testcover/runner.go:250`

**Issue:** The one new row, `ElapsedRunsFromTheRunLineToTheResultLine`, prints both lines in one `Printf` (`testmain_test.go`, `standInTimed`).
A real test binary's output reaches the pipe in arbitrary chunks and holds indented subtest result lines, and the `scanned` bookkeeping and the `HasPrefix("--- ")` filter that handle them are pinned by nothing.

### [LOW] The empty-contained answer's unconfirmed key is unasserted

**Location:** `internal/gatecli/redundant.go:234`

**Issue:** The only assertion on `unconfirmed` is in the contained path of `TestGateRedundant`; the early return's new key could be dropped with no test failing.

## Rejected

- code-tests LOW, refuted-over-unconfirmed precedence in `redundant.go:285-293` unpinned in gatecli: `TestJudgeMutants` row "a covering run that was unjudged confirms no kill, yet a unique kill still refutes" pins the verdict that decides it, and `TestGateRedundant`'s refuted `TestClassifyChecked` pins the `Unique > 0` branch; a further gatecli case would duplicate them.
- Board entry's funcs-order part (focus item 1): not unmet. `fillMapTests` sorts a new run's blocks with `slices.Sort` and its funcs with `slices.Compact(slices.Sorted(...))`, a kept entry's through `reindexed`, and the function table by `CompareFuncKeys` (`internal/gatecli/testmappackage.go:625-670`), so the order the entry asks for already holds and the summary's account of it is accurate.
- Trimmed `Long` help of `lym gate redundant` (commit `8b745a81`): dropping the operator list and the removable-set rule was forced by the help length cap once the unconfirmed sentence went in, and both facts stay in `internal/gatecli/doc.go` and `internal/testcover/doc.go`.

## Focus departures

- "`internal/testcover/runner.go` per-test seconds: settle whether the RUN-line to result-line time is correct for subtests, parallel tests, a test with no RUN line, and output split across reads": settled by reading the code (correct in each case, per the code-correctness lens), not by running the gate; the untested cases are F5.
- `lym board get redundant-timeout-not-kill` failed with an invalid-json error, so the board entry was read from the decision record's `## Goal` fenced body only.
