# Test suite: cut change amplification

`test-suite-change-amplification` · Planned · feature

## Goal

A small production change costs edits in few test files, so an implementing agent reads less test context per change, and integration tests leave no tmux sockets behind.
Runs alone: it touches test files in nearly every package, so a concurrent run would conflict.
It runs first in Layer A, so the tasks after it change fewer test files, and before `pattern-init`, so its Testkit Invariant lands in `CONSTRAINTS.md` before that file moves.
#326 section 2 needs an operator decision before Discussion closes.

## Survey and work items

Survey of main at fafb660c3, 2026-10-01: test churn 1.68x prod churn and rising; one loom-recipe row costs edits to ~8 test files in 4 packages; worst packages cmd/lyx, loomrecipe, fabricengine.
Work items, each self-contained with file:line evidence in its issue:
- #324 shared `internal/testkit` and one fake per cross-package seam (reverses the do-not-share-test-helpers decision; needs a CONSTRAINTS.md Testkit Invariant).
- #325 git helpers in gitkit, hubforge as the only fabric fixture (84 private git helper copies today).
- #326 one invariant-scan harness; derive registry and CLI-tree tests (section 2 needs an operator decision: it changes the CLI/Cobra Invariant help-tree tests).
- #327 decouple recipe, plan and stencil tests from their sources.
Ties to convergence-plan: real-wiring tests over per-package fakes.

## From `test-tmux-socket-leak`: Integration tests leave tmux sockets behind

Tests leak tmux sockets: `/tmp/tmux-1000/` holds hundreds of `lyx-Test…`, `lyx-contract-…` and `lyx-warp-bare-…` sockets left by reed/loom integration tests. Test fixtures should kill their tmux server and remove the socket in `t.Cleanup`.


