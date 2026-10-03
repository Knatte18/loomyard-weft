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

## Operator directive: CONSTRAINTS.md may change

The operator suspects some `CONSTRAINTS.md` invariants, or the way they are enforced, are what made the test suite grow, and authorizes changing them.
Discussion audits the invariants that bear on this task's tests and, for each one that forces duplicated tests or hand-kept copies (per-package copies of the same scan, per-module test obligations, allowlists that mirror code, the do-not-share-test-helpers decision), proposes a change: restate the rule, move its enforcement to one shared scan, or drop it.
A changed invariant lands in `CONSTRAINTS.md` in the same commit as its tests; an invariant's intent is weakened only where the discussion says so explicitly and why.

## Notes from part 1's invariant audit (its decision D13)

- The per-package `seam_enforcement_test.go` and `leaf_enforcement_test.go` copies share root lookup, walk, allowlist and floor scaffolding: #326's one-harness item, which can host part 1's Testkit scan too (part 1's scan does not assume its own shape is final).
- Tier Purity's and Hermetic Env's per-file allowlist entries whose only reason is resolving the module root through `go env GOMOD` mirror each guard's implementation; one shared root resolver in the harness retires them.
- Hand-kept copies of registries production owns: CLI / Cobra Invariant help-tree tests, Sandbox Suite Coverage, the config-section registry, Cwd Resolution's constructor-anchoring and no-transients path mirrors, Shed Recipe Registry's row mirrors, Agent Name and Stencil Ownership tests.
