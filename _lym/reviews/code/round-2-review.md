---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/redundant_integration_test.go:159
    summary: TestGateRedundant waits out real 10 s mutant timeouts, about 41 s per run, because redundantCmd hard-codes secondUnit to time.Second and the test drives the verb through Command(), breaking PATTERN-test-speed's Time is injectable rule
    fix: In internal/gatecli/redundant_integration_test.go, drive the ListsTheCandidates… subtest through runRedundant directly with a redundantRequest whose secondUnit is short, the way testmap_integration_test.go:573 and :1385 pass 150 ms to runTestMap. Pass the seam cwd through the context as those tests do, set goBinary to the production default and pass the environ, minuteUnit and grace that Command() passes. Pick a unit that keeps the mutantTimeoutFloor bound, mutantTimeoutFloor × secondUnit, far above a healthy fixture test's whole process run on a loaded machine. Raise the fixture's redundancy_test_timeout_sec so that the test bound, redundancy_test_timeout_sec × secondUnit, lies well above that floor. The endless Tally mutant then ends at the floor, not the test bound, which also makes the fixture pin that mutantTimeout's floor binds below the test bound. Keep redundantCmd passing time.Second in production. Update the comment above the gate string to name the new bounds.
  - id: F2
    severity: LOW
    class: design
    location: internal/testcover/sourcescan_test.go:68
    summary: TestScanSourceReaders_MarksTestsReachingTheScanKit relies on random map iteration order over 16 calls to reach both walk orders of the call cycle, so a regression in one order is caught only with high probability, not deterministically
    fix: In internal/testcover/sourcescan_test.go, make each walk order its own row of a table-driven test instead of looping 16 times. One row writes a file whose only Test function is TestKitBehindCycleFromOuter and wants it marked. Another row has TestKitBehindCycleFromInner alone, and a third has both tests and wants both marked. Each row writes its own t.TempDir package, so the walk starts from one known test. Keep the other tests, TestDirect, TestImportedFunction, TestThroughHelper, TestThroughMethod, TestNoScan and TestCycle, in a row that holds no cycle reaching the kit. Drop the for range 16 loop and its comment.
---

### [BLOCKING] TestGateRedundant waits out real production-second timeouts

**Location:** internal/gatecli/redundant_integration_test.go:159

**Issue:** `PATTERN-test-speed` § Time is injectable says no test in any tier waits out a production duration and that the test passes a short one.
`redundantRequest` carries a `secondUnit` field, but `redundantCmd` (internal/gatecli/redundant.go:114) always sets it to `time.Second`, and `TestGateRedundant` runs the verb through `Command()`.
The fixture's dropped `n -= 1` mutant makes `Tally` endless.
Its runs end only at `min(mutantTimeoutFloor × secondUnit, redundancy_test_timeout_sec × secondUnit)`, which is 10 real seconds.
That happens in the candidate stage, in the side-by-side cover stage and again in each serial alone rerun in `runJobs`.
I ran `lym gate test --tier 1 ./internal/gatecli -- -run 'TestGateRedundant$'`.
The subtest `ListsTheCandidatesMutationConfirmsAndSetsApartTheRefutedKeptAndUnjudgedOnes` took 40.97 s, almost all of it spent waiting out those timeouts.
The fixture also sets the test bound equal to the floor, so it cannot show which of the two bounds `mutationCheck.mutantTimeout` applied.
This round's focus asks whether a test reaches the mutant timeout bound.
It reaches it, but only at a value the test bound alone would also give.

### [LOW] The source-scan test samples a random order instead of pinning each order

**Location:** internal/testcover/sourcescan_test.go:68

**Issue:** The fix for the call-cycle memo depends on the walk order: the answer differs depending on which member of the cycle is walked first.
`ScanSourceReaders` walks `src.funcs`, a map, so the test calls it 16 times and relies on Go's randomized iteration to reach both orders.
The testing rules forbid random dependence.
As written, a regression in one order fails the test only with probability 1 − 2⁻¹⁶ rather than every time.
Each order can be reached deterministically by giving the scanned package only one of the two cycle tests.

On the rest of this round's focus:

- `runRedundant` prints `coverings[j]`, which `mutationCheck.run` takes straight from each `MutantCandidate.Covering`.
  That field is the `testcover.CoveringTests` pick that `JudgeMutants` judges against, so the printed list and the judged list are the same.
  `TestGateRedundant` compares the `Covering` lists exactly through `cmp.Diff`.
- In `runMutants`, a cover's kill is queued, cleared from `kills` and rerun through `runJobs(…, true)`, so a failure seen beside other runs is decided by a run alone.
  This includes a cover that is also a candidate and killed the mutant in its candidate run.
  A cover that ran the mutant as a candidate and passed keeps that pass, which is sound.
- The `cov` fixture reaches the drop-and-rebuild loop: `Timeout`'s `5` behind `lymMutantInt` does not compile, and the test asserts `Unbuildable: 1`.
  It reaches the `//lym:guard` exclusion: `TestGuarded` is listed as not judged with `ReasonSourceCheck`.
  It counts a timed-out run as a kill: each Tally test's `MutantsKilled: 7` includes the endless-loop mutant.
  The timeout bound is covered by F1.
- `ScanSourceReaders` memoizes a false answer only once no function above it on the call path is involved, so a cycle settles correctly from either entry.
  It does not read a selector call on an imported package as a method call.
  `TestImportedFunction` and the two `TestKitBehindCycle…` rows pin these cases, and `TestCycle` pins a cycle with no kit, subject to F2.
