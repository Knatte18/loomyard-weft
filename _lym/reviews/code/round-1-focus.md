---
round: 1
focus:
  - "internal/gatecli/run.go, internal/gatecli/cli.go: does --stop-on-fail halt at the first failing tier, and does `lym gate test --upto` keep its default of running every tier (the full report stays a separate choice)?"
  - "internal/gateslot/runlog.go, tiers.go (FailedTestsCall): does the failed-test rerun read the last run's recorded failures correctly, and what path does a failure with no failing test (crashed runner, build error) take, per the Goal's separate-path requirement?"
  - "internal/loomshed/verifygate.go: between attempts, does only the failed-tests rerun go to the fixer, and does the full gate run once after the rerun passes? Check the attempt count and the loop exit on both paths."
  - "internal/loomshed/verifybudget.go and internal/loomcli/darnwiring.go: is the budget notice sent once, to the orch, with the failures to date? Is the choice of notice over stop or hand-off consistent across the recipe, darn.yaml and darn config (verify_budget_min), and is a zero or unset budget handled?"
  - "internal/shuttleengine/gate.go, run.go, wait.go and internal/shedadapters/gateattempts.go: does the halt message for a step stopped with no gate attempt run state why, and does the reported attempt count match the attempts made?"
  - "Claims in _lym/landing/summary.md against the diff, and the repository's own rules: pattern/PATTERN-gate-slot.md change, doc.go updates, and config registration tests (configreg, envkit)."
  - "Test economy: loomshed/verifygate_rerun_integration_test.go, verifybudget_test.go, gateslot/failedtests_test.go and gatecli/test_integration_test.go against the tests that already cover the same behavior."
---

Seed focus from the decision record's four changes: stop at the first failing tier, rerun only failed tests, a time budget, and the zero-attempts halt message.
