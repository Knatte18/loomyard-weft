---
round: 1
focus:
  - "internal/gatecli/redundancy.go: does the step never change the gate's exit result, turning every build, test, map, merge-base and lym-build error into a report line, and log a failed report write at Warn?"
  - "internal/gatecli/redundancy.go: is the gate slot taken after the key's tiers release theirs and held to the step's end, with the budget counted from slot acquisition, no build or test started once spent, and every unrun test, including whole unbuilt packages, listed as not judged?"
  - "internal/gatecli/redundancy.go and redundancyreport.go: does the skip-on-match check use the current files rather than HEAD, and still rerun when the report holds an error line, a package with no map or failed-build map, or a test past the budget, including when the diff also changes a tier2 test?"
  - "internal/gatecli/redundancytests.go: does test selection look up each current top-level test by package and name at the merge base through CodeDirAt and CodeFileAt, leaving renamed or moved files' unchanged tests untaken, skipping deleted files, and judging nothing with a stated reason when there is no parent branch or the scoper is not go?"
  - "internal/gatecli/redundancy.go: are covers the map tests with unchanged function hash minus the judged test itself plus the other judged tests, so a judged test is never its own cover and two judged tests covering only each other are never both contained?"
  - "internal/testcover (blockkey.go, testfuncs.go, verdict.go): does the greedy pass over judged tests with every map test kept behave as D7 step 5 asks, and do the changes to existing functions keep their prior callers and behavior intact?"
  - "internal/gateslot (config.go, template.yaml, tiers.go): does the reports list refuse an unknown name with the known names as the way forward, carry redundancy_max_tests, redundancy_budget_min and the timeouts with the stated defaults, and place reports: [redundancy] on webster, parent-align, darn-write and code-burler-fix so a tuned base entry without reports gets it and reports: [] turns it off, with a test reaching each?"
  - "Test economy across redundancy_integration_test.go, config_test.go, blockkey_test.go and verdict_test.go: any duplicate coverage, and any acceptance criterion in the decision record the tests leave uncovered; do the tests avoid launching a real test binary or go test?"
  - "Drops and cap in redundancytests.go: are tier2, tier3, undeclared-tier and testtiming:keep tests dropped with their reasons, and are the first redundancy_max_tests kept in file then name order with the rest listed as not judged?"
---

Focus covers the decision record's D7 steps and acceptance criteria, ordered by risk: the never-fails-the-gate invariant, slot and budget handling, skip-on-match, and cover construction.
