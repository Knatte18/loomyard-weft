# Fixer-report, round 1

## Changes

- F1 (LOW): added the `failures` row "the last error envelope wins and an ok envelope is skipped" to `TestStartCapture` in `internal/driftengine/spawn_test.go`; fixed as proposed.
- F2 (LOW): `runCaptureChild` in `internal/driftcli/cli.go` marks `LastingFaultKey` only for `lymcwd.ErrNotAGitRepo` and `lymcwd.ErrCwdOutsideAnchor`; any other `openWorktree` error takes `fail(err)`. Fixed as proposed. `internal/driftcli/doc.go` now names those two refusals, which the self-check found it over-claiming.
- F3 (NIT): the overlong-error row repeats `å` instead of `x`, so only a rune-based cut passes; fixed as proposed.

Commits: `driftcli: fix F2` (cli.go), `driftengine: fix F1, F3` (spawn_test.go; both findings touch the same file and landed in one commit), `driftcli: fix F2` (doc.go).
`lym gate test ./internal/driftengine ./internal/driftcli` passed at tier 0 after the code and test edits (197 passed, 0 failed); the doc.go edit is a comment only.

## Self-check

- fix-consistency, LOW: `driftcli/doc.go:20` over-claimed the lasting mark after the narrowing; fixed as proposed (reworded).
- fix-proposal, LOW: the unresolvable-cwd case (`CwdFrom`, non-sentinel `Resolve` errors) is no longer lasting; the same doc over-claim, fixed by the doc.go reword. The narrowing itself follows F2's `fix` key and stays.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
