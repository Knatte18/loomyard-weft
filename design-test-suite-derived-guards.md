# Test suite, part 2: derived guards, decoupled recipe tests, no leaked tmux sockets

`test-suite-derived-guards` · Planned · feature

Depends on: [`test-suite-change-amplification`](design-test-suite-change-amplification.md)

## Goal

Tests derive their expectations from the code they guard instead of keeping hand-written copies of lists owned elsewhere, so adding a module, command, config section, path or recipe row edits no test file; and integration tests leave no tmux sockets behind.
Second half of the test-suite cleanup: it builds on `test-suite-change-amplification`'s `internal/testkit` (the scan harness lives under it).
#326 section 2 needs an operator decision before Discussion closes.

## Work items

Each self-contained with file:line evidence in its issue:
- #326 one invariant-scan harness; derive registry and CLI-tree tests (section 2 needs an operator decision: it changes the CLI/Cobra Invariant help-tree tests).
- #327 decouple recipe, plan and stencil tests from their sources.

## From `test-tmux-socket-leak`: Integration tests leave tmux sockets behind

Tests leak tmux sockets: `/tmp/tmux-1000/` holds hundreds of `lyx-Test…`, `lyx-contract-…` and `lyx-warp-bare-…` sockets left by reed/loom integration tests. Test fixtures should kill their tmux server and remove the socket in `t.Cleanup`.
