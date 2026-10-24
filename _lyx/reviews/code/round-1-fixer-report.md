# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): `firstRetiredTierSpelling` in `cmd/lyx/tiertags_test.go` walks the `constraint.Expr` tree instead of calling `Eval`.
  Added the `tier1 && integration` row.
  The `linux && tmux` row the review also named was dropped: it fails the file through the multi-tier admission check even with the old `Eval` closure, so it pins nothing (self-check, fix-code-correctness).
- F2 (fixed as proposed): the guard-tag bullet in `pattern/PATTERN-test-speed.md` now says a guard file takes its one `tierN` tag and the scan refuses the retired spellings.
- F3 (fixed as proposed): `pattern/PATTERN-test-isolation.md` respelled at the lines the review named.
- F4 (fixed as proposed): `contracts/specs/loom-plan-spec.md` no longer lists the older spellings.
- Self-check follow-ups, in a second commit:
  - respelled the remaining retired tags in `pattern/PATTERN-test-speed.md`, `pattern/PATTERN-test-economy.md` and `pattern/PATTERN-testkit.md`;
  - respelled the comments and the two finding messages in `internal/testkit/tmuxkit/tmuxkit.go` and `allow_none.go`, which told authors to tag a file `tmux`;
  - `mentionsIsolationTag` in `cmd/lyx/tmuxisolation_test.go` walks the tree like F1's function.

`lyx gate test --tier 0 ./cmd/lyx ./internal/testkit/tmuxkit` is green (the first run died on a `newosproc` thread-creation failure; the identical rerun passed).

## Self-check

- fix-code-correctness, LOW, redundant `linux && tmux` row: fixed, row removed.
- fix-consistency, MEDIUM, `PATTERN-test-speed.md:70` contradicts the F2 bullet: fixed, respelled.
- fix-consistency, MEDIUM, `PATTERN-test-speed.md` lines 42, 49, 61, 103: fixed, respelled.
- fix-consistency, LOW, `PATTERN-test-economy.md:24`, `PATTERN-testkit.md:15`: fixed, respelled.
- fix-consistency, LOW, `tmuxkit.go:19` comment stale against the PATTERN respell: fixed, respelled.
- fix-affected-set, LOW, retired spellings across `PATTERN-test-speed.md`: fixed, same change as above.
- fix-affected-set, LOW, `PATTERN-test-economy.md:24`, `PATTERN-testkit.md:15`: fixed.
- fix-affected-set, LOW, `PATTERN-test-isolation.md` remaining hits: no change, they are tmux binary and socket-directory names, not tags.
- fix-proposal, MEDIUM, `tmuxkit.go` messages tell authors to tag `tmux`: fixed, messages and comments respelled to `tier2`/`tier3`; no test pins the message text.
- fix-proposal, LOW, `mentionsIsolationTag` shares the `Eval` short-circuit: fixed with a tree walk.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
