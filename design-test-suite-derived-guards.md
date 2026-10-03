# Test suite, part 2: derived guards, decoupled recipe tests, no leaked tmux sockets

`test-suite-derived-guards` · Tasks · feature · done

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
- Plan fixtures: part 1's parent review moved its proposed `internal/testkit/plankit` here, as #327 section 2's plan-fixture writer; it also covers the glyph-plan and file-tree fixture copies in `loomshed` and `loomcli` that part 1's discussion had found.

## Socket leak: findings and direction (ly:orch, 2026-10-03)

`/tmp/tmux-1000/` held about 1,500 dead socket files (656 `lyx-warp-…`, 415 `lyx-001…`, 115 `lyx-contract-…`, the rest per test name) with no tmux server behind any of them: servers exit, their socket files stay.
About 34 tests already run `kill-server` in `t.Cleanup`, which ends the server but leaves the file; others, likely the fabric tests that boot reed (`lyx-warp-…`), have no cleanup at all.
Direction for Discussion to weigh:
- One shared tmux fixture (under `internal/testkit`) for every test that needs a server: a unique socket, `kill-server` and removal of the socket file in `t.Cleanup`.
- The socket under a short per-test temp directory (`tmux -S <path>`, not `-L <name>`), so it goes with the directory even when a test dies; mind the ~108-byte Unix socket path limit, which long `t.TempDir()` paths can exceed.
- A guard: each tmux-using package's `TestMain` compares the socket directory before and after its tests and fails on a new leftover `lyx-*` socket.
