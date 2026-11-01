# Fixer-report, code round 1

## Changes

- F1 (fixed as proposed, commit `1aa81e1b`): `contracts/specs/shed-recipe-spec.md` lists `clean`, `discussion`, `plan`, `parts`, `description` and `verify`, and states that `clean` fails while git reports an uncommitted or untracked path and requires the absolute `Env.WorktreeRoot`.
- F2 (fixed as proposed, commit `f513b8ce`): `TestResolveGateSpec_CleanRequiresAbsoluteWorktreeRoot` in `internal/shedrecipe/entries_gate_test.go` refuses an empty and a relative `WorktreeRoot` for the `clean` gate and asserts the error names `WorktreeRoot`.
- F3 (fixed as proposed, commit `39a0b2b1`): a `TestDarnVerifyBudget` subtest in `internal/loomrecipe/darnrecipe_test.go` asserts the tapestry `Code-Burler-Fix` row's gates are `clean` then `verify`.

`lym gate test ./internal/shedrecipe ./internal/loomrecipe` passed with no failures.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
