```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that pins it, or none.

**Per behaviour, the test that pins it**

1. `runMutants`' `record`: a timed-out or memory-capped run goes to `timedOut`/`capped` and no longer into `kills`.
   Pinned by `TestGateRedundant` → `ListsTheCandidates…Unconfirmed…` (the `Unconfirmed` rows, the `MemoryCapped`/`TimedOut` stops, `coveringRuns = 14`).
2. `runMutants` returns `unjudged`, built from both `timedOut` and `capped`.
   Pinned by the same subtest: the Hoard tests (capped) and the Tally tests (timed out) are both listed with `Unjudged`.
3. `runMutants` requeue `!timedOut[run]`: a covering test that is itself a candidate and timed out on the mutant as a candidate reruns alone, and its stop is cleared by `delete(timedOut, run)` before the rerun.
   **None.** In the fixture no candidate kills `tallyStep` or `hoardLoop`, so no covering run of those mutants is ever queued (`redundant_integration_test.go:300-302`), and the branch is never reached.
4. A covering run, rerun alone, that itself times out or hits the cap is recorded into `unjudged`.
   **None at the `runMutants` level.** The fixture never queues such a run, as in item 3.
   At the `JudgeMutants` level it is pinned by row 2.
5. `JudgeMutants`, own run unjudged: the mutant goes to `Unjudged`, the candidate is neither redundant nor kills-none.
   Pinned by `TestJudgeMutants` row "a candidate whose own run of a mutant was unjudged is unconfirmed…" (`A` and `Stalls`).
6. `JudgeMutants`, a covering run unjudged and no covering kill: the mutant goes to `Unjudged` and not `Unique`, and a unique kill still refutes.
   Pinned by row "a covering run that was unjudged confirms no kill, yet a unique kill still refutes" (`A`, `C`).
7. `JudgeMutants`: `ReasonKillsNone` only when nothing was killed and nothing was unjudged.
   Pinned by `Stalls` in row 1, and by `TestDoubleUnchecked` in the integration test.
8. `redundant.go` switch: redundant, then `Unique&gt;0` → refuted, then default → unconfirmed with `MutantsKilled` and `Unjudged`.
   Pinned by the integration subtest (`TestClassifyChecked` refuted; the four unconfirmed rows, with `MutantsKilled` 0 and 6).
   The precedence of refuted over unconfirmed for a candidate holding both is **not pinned in gatecli**: no listed test there has both.
9. `mutantSites`: pinned by the integration subtest's `Refuted` and `Unconfirmed` rows.
10. The empty-contained early return now emits `"unconfirmed": []`.
    **None.** Grep finds `unconfirmed` asserted only in the one integration subtest, which takes the contained path.
11. `testOutput`: elapsed time runs from the RUN line to the result line, both arriving in one write.
    Pinned by `TestRunAlone/ElapsedRunsFromTheRunLineToTheResultLine` (`runner_test.go`, `standInTimed`, stepping clock giving 0.0015).
12. `RunAlone` keeps the binary's own figure when the output has no RUN line.
    Pinned by `PassWithNoProfileFlag` and the other 0.25 rows: `standInPass` prints no RUN line (`testmain_test.go:143`).
13. `testOutput.Write` assembling a line split across several writes, and `stamp` ignoring an indented subtest result line or another test's result line.
    **None.** Every stand-in prints each line within one `Printf` (`testmain_test.go:143-193`).
14. `Toolchain.Now` nil defaults to `time.Now`.
    Pinned indirectly: every other `TestRunAlone` row passes a nil `Now` and would panic without the default.

**Findings**

- **MEDIUM**, design. Location: `internal/gatecli/redundantmutants.go:298` and `:307-312`.
  - Summary: the new requeue of a covering test that timed out on a mutant as a candidate beside other runs, and the clearing of its stop before the rerun, have no test that reaches them. Neither has the recording of a covering rerun that itself times out or hits the cap.
  - Evidence: `redundant_integration_test.go:300-302`. No candidate kills the loop mutants, so no covering run of them is queued. `TestRunMutantsGuardKilledRunIsNoKill` (`redundantmutants_guard_test.go:76`) discards `unjudged`.
  - Fix: add a case where one candidate kills a mutant that a candidate covering it times out on beside other runs, then assert the alone rerun and the resulting kill or unjudged entry.
- **MEDIUM**, design. Location: `internal/testcover/runner.go:250-276` (`testOutput.Write`, `stamp`).
  - Summary: no test pins a line split across writes, nor that an indented subtest result line or another test's result line does not end the timing.
  - Evidence: `runner_test.go` row `ElapsedRunsFromTheRunLineToTheResultLine`. `standInTimed` writes both lines in one `Printf` (`testmain_test.go:148`).
  - The landing summary's claim that the expected value "holds however the output is chunked" is untested.
  - Fix: one stand-in row that writes the RUN line, a subtest's indented `--- PASS` line, and the result line split mid-line across separate writes.
- **LOW**, design. Location: `internal/gatecli/redundant.go:234`.
  - Summary: the empty-contained path's new `"unconfirmed": []` key is asserted by no test.
  - Evidence: a grep of `internal/gatecli` tests finds `unconfirmed` only in `redundant_integration_test.go:236-337`, the contained path.
- **LOW**, design. Location: `internal/gatecli/redundant.go:285-293`.
  - Summary: a candidate with both a unique kill and an unjudged mutant must land in `refuted`, not `unconfirmed`. `TestJudgeMutants` pins the verdict (row 2, `C`), but no gatecli test pins the switch's precedence.
  - Evidence: the integration subtest's refuted `TestClassifyChecked` has no unjudged mutant (`redundant_integration_test.go:338-345`).

Tool calls:
1. Grep of `testmain_test.go` print lines and gatecli tests for `unconfirmed`. My context lacked which stand-ins print a RUN line, and whether any test asserts the empty-contained answer.
