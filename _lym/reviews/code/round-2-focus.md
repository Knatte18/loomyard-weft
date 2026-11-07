---
round: 2
focus:
  - "In internal/gatecli/tierrun.go, runInvocations expands {test_targets}; settle whether a test pins that a command holding {test_targets} receives only the packages holding a test of the tier, that {targets} stays whole, and that when every package is dropped no command runs, a stderr line is printed and the exit is 0."
  - "In internal/impactset/tiertests.go, settle whether a package whose only tests are ExampleXxx functions with an Output comment still keeps its {test_targets} slot."
  - "In internal/gatecli/testtargets_test.go, settle whether tierTestTargets' go-list failure fallback and a \"./...\" pattern target are each covered by a test row."
  - "Across internal/gateslot/doc.go, internal/gatecli/tiers.go, internal/gatecli/doc.go and internal/gatecli/run.go, settle whether every doc, comment and refusal message about the all_targets rule and the guard step names {test_targets} wherever HoldsTargets accepts it."
---
