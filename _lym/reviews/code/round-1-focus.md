---
round: 1
focus:
  - "internal/battencli/lifecycle_integration_test.go: the groups of TestBattenIntegration_Rows now run in parallel. Read every step function and its helpers for state wider than one hub (package-level vars and swapped seams, environment, working directory, process-wide registries, fixed paths or names outside the hub): can two groups interfere or race?"
  - "internal/battencli/lifecycle_integration_test.go: each step now starts on a hub holding only its own group's earlier steps. Does any step rely, for its setup or for an assertion to mean something, on state a step now in another group used to leave behind, so that it fails or passes vacuously?"
  - "internal/battencli/lifecycle_integration_test.go: the step order within and across groups changed. Does every step that changes its hub for good run last in its group, and does the new table hold each old step exactly once?"
  - "stepStartCost_CommandRowsCostUnderTheBoundBeyondTheCommitAndTheProcess asserts a wall-clock bound and now runs while other groups load the machine. Does the bound still hold what its name and the testtiming:keep line state, and can parallel load trip it?"
  - "_lym/landing/summary.md, Measurements: do the command, tags and tier measured match what the board entry's Aim refers to (33 to 75 s tier1 against figures near 10 s)? Is the stated reason for the gap supported, is any set-up difference named, and do the table and the prose after it agree, including the longest-test claim and the per-group balance claim?"
  - "pattern/PATTERN-pair-remote.md and the rest of the tree: does any doc, comment, spec or tool configuration still name a TestBattenIntegration_Rows subtest by its old path, or describe the steps as serialised on one shared hub?"
---

Seed pass for round 1.
The diff regroups one integration test to run in parallel and updates one subtest path in a pattern file,
so the entries aim at cross-group isolation, hidden order dependencies, the timing-bound step and the reported measurements.
