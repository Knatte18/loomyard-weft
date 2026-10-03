# Test suite, part 1: shared testkit, seam fakes and git helpers

`test-suite-change-amplification` · Planned · feature · running · Discussion-Write

## Goal

A small production change costs edits in few test files, because test helpers and seam fakes live in one place instead of being copied per package.
This is the first half of the test-suite cleanup; `test-suite-derived-guards` is the second and builds on this one's `internal/testkit`.
It lands its Testkit Invariant in `CONSTRAINTS.md` before `pattern-init` moves that file.

## Survey and work items

Survey of main at fafb660c3, 2026-10-01: test churn 1.68x prod churn and rising; one loom-recipe row costs edits to ~8 test files in 4 packages; worst packages cmd/lyx, loomrecipe, fabricengine.
Work items, each self-contained with file:line evidence in its issue:
- #324 shared `internal/testkit` and one fake per cross-package seam (reverses the do-not-share-test-helpers decision; needs a CONSTRAINTS.md Testkit Invariant).
- #325 git helpers in gitkit, hubforge as the only fabric fixture (84 private git helper copies today).
Ties to convergence-plan: real-wiring tests over per-package fakes.

## Out of scope

#326, #327 and the tmux socket leak belong to `test-suite-derived-guards`.
