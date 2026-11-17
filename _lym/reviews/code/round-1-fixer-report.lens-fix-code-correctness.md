```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness through the fix-code-correctness lens.

Findings:

1. **MEDIUM** — four new comment lines run to 120 characters or more. The rule is under 120, and the repo's comment lint and the earlier commit `0d1b952f` enforce it. Measured on the added comment text:
   - `internal/gatecli/doc.go`, the new `unconfirmed` line ("its own run of a mutant none of its covering tests killed, or …"): 133.
   - `internal/gatecli/redundant_integration_test.go`, "Each covering test then runs with each mutant a contained test killed or left unjudged, when it covers the mutant.": 120, which is over the limit by the "119 passes" rule.
   - `internal/testcover/mutantverdict.go`, the `JudgeMutants` line "A candidate's own unjudged run of a mutant a covering test killed leaves that mutant out of its Killed, Unique and Unjudged:": 124.
   - `internal/testcover/testmain_test.go`, the `runStandIn` doc line about `standInSplit`: 144.

   Each needs a break at its comma or colon.

2. **LOW** — the F5 test does not reliably fail with the defect in place (`internal/testcover/testmain_test.go` `standInSplit`, `runner_test.go` row `ElapsedStampsOnlyTheRunLineAndTheCompletedTopLevelResultLine`).
   - `testResultLine` is `(?m)^--- …` (`runner.go:153`), so the indented `    --- PASS: …/sub` line never matches. Removing the `HasPrefix("--- ")` filter at `runner.go:271` therefore breaks nothing, and the filter stays unpinned.
   - The mid-line split only reaches `testOutput.Write` as separate chunks if the pipe reader happens to read between the stand-in's writes. If it reads both chunks together, the row passes with a per-write stamping bug. This is the case I traced: a bug that stamps partial lines would leave `ended` unset, which fails the row only when the chunks arrive apart.
   - The `scanned` bookkeeping is not pinned either. If `scanned` never advanced, `started` and `ended` are each stamped once, so Elapsed is unchanged.

3. **LOW** — in `JudgeMutants` (`internal/testcover/mutantverdict.go`), a candidate whose every mutant is own-unjudged but killed by a covering test ends with empty `Killed` and `Unjudged`. It then gets `ReasonKillsNone` ("kills no mutant of its blocks") and is listed as not judged. That reason is misleading, since the candidate's own run was unjudged, and nothing pins the case. It follows from F1's proposed change, so this is a note.

Traced with no defect found:
- **`runMutants` queue loop:** with B timed out as a candidate of mutant 1 and A having killed it, B is requeued.
  - Its `timedOut` entry is cleared before the rerun.
  - The rerun's kill is recorded, and `unjudged` no longer holds B.
  - A capped candidate run is still skipped, as intended.
  - Where a candidate's own run was unjudged and its covering test killed beside other runs, that covering test is requeued and reruns alone.
- **`TestRunMutantsRerunsACoveringTestThatTimedOutAsACandidate`:** it fails if the `!timedOut[run]` clause is reverted, because B is then never requeued and stays unjudged. It also fails without `delete(timedOut, run)`, because the stale timeout survives the rerun's kill.
- **F6 subtest:** dropping the `unconfirmed` key leaves an empty `RawMessage`, which is not `"[]"`, so the subtest fails.
- **`coveringRuns = 16`:** two tally-loop covering runs and their reruns add 4 runs, and the capped hoard runs are skipped. The tier1 gate run passed with this value.

Tool calls (2):
- `git diff 8b745a81 HEAD -U0 -- '*.go'` filtered by awk: measured the length of every added comment line. My context lacked the final committed text of those lines.
- (An earlier gate run in my context gave the pass result I rely on; I made no other reads.)
