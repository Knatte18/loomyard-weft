---
round: 1
focus:
  - "internal/impactset/tiertests.go: does the derivation of packages holding a tier's tests (from tiergate.Require calls) match how tests are really assigned to a tier, including tests in external _test packages, helper-based Require calls, build-tagged files, and packages with no Require call? Settle whether a package with a real test of the tier can be dropped from {test_targets}."
  - "internal/gatecli/tierrun.go and tiers.go: how {test_targets} is filled per invocation. Settle the empty case (no package holds a test of the tier: does go test get no arguments and run ./... or the whole scope?), targets that are patterns or paths, and that {targets} stays whole for build and vet."
  - "internal/gateslot/tiers.go, template.yaml, config_test.go: does the new placeholder validate, and do existing configs that use only {targets} keep working? Settle whether the goal's 'applies to every tier, not tier 1 alone' is met, i.e. the template command for every tier, not only tier 1, uses {test_targets}."
  - "internal/gatecli/green.go and tiers_unix_test.go: do the edits change green-state or caching behavior beyond the stated goal? Settle whether the diff licenses each change against the board entry."
  - "Scope against the board entry: the entry asks for a derived list so a package gaining its first test of a tier enters without a hand edit. Settle that nothing is hand-maintained, and that pattern/PATTERN-test-speed.md changes only what the entry licenses."
  - "Test economy: testtargets_test.go, tiers_test.go and impactset/tiertests_test.go. Settle whether any new test duplicates another's coverage, and whether any added behavior (the empty case, error paths from reading packages) has no test."
---

Seed focus from the Goal in the decision record alone (its other sections are empty) and the diff stat.
