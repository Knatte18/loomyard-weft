# Test suite: cut change amplification

`test-suite-change-amplification` · Someday · feature

Survey of main at fafb660c3, 2026-10-01: test churn 1.68x prod churn and rising; one loom-recipe row costs edits to ~8 test files in 4 packages; worst packages cmd/lyx, loomrecipe, fabricengine.
Work items, each self-contained with file:line evidence in its issue:
- #324 shared `internal/testkit` and one fake per cross-package seam (reverses the do-not-share-test-helpers decision; needs a CONSTRAINTS.md Testkit Invariant).
- #325 git helpers in gitkit, hubforge as the only fabric fixture (84 private git helper copies today).
- #326 one invariant-scan harness; derive registry and CLI-tree tests (section 2 needs an operator decision: it changes the CLI/Cobra Invariant help-tree tests).
- #327 decouple recipe, plan and stencil tests from their sources.
Ties to convergence-plan: real-wiring tests over per-package fakes.