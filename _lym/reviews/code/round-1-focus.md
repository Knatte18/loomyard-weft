---
round: 1
focus:
  - "internal/gatecli/testmap_integration_test.go: trace every path by which a healthy fixture test can still reach passed_alone, timed_out or failed_tests under load (the --all call that sets testMapHangingEnv, calls with their own envs or contexts, a non-timeout failure of the stand-in), and settle whether the diff removes the cause of the reported flake in each subtest that maps alpha, as the amended entry and the parent's 'deterministic' decision ask."
  - "Same file: for every subtest and helper that maps alpha or sets testMapHangingEnv, including WhileATestHangs and the later subtests that read alpha's map or compare exit codes, settle whether it still checks what its name and comments state now that the HangsOnCue test passes without the cue, and whether any state carried between subtests relied on the timed-out entry."
  - "Same file and the sibling gatecli integration tests: the 500 ms testMapSecondUnit also governs other durations derived from secondUnit (mutant, build and call bounds). Settle which limits and waits it widens, whether any surrounding context deadline or test wait is now too tight, and whether any map call still runs under the old short unit."
  - "Commit trailers (Root-Cause, Loosened-Check) and _lym/landing/summary.md against the code: check that the stated cause and cited measurement are borne out by the issue's log, and that the claims about wait counts, which calls wait out a limit, and which suffix users are unchanged are true."
---

Seed focus for round 1.
The diff is confined to one test file, so the questions centre on whether the flake is removed and whether the changed fixtures still pin their stated behavior.
