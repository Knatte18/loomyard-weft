# Fixer-report — code review round 1

## Changes

- F3 (NIT), commit 1a9a08f83: `internal/gatecli/scope.go` sorts the tag tiers with `slices.Sort` and drops the `sort` import. Fixed as proposed.
- F1 (LOW) and F2 (LOW), commit d6209f3c1, both in `internal/gatecli/scopes_integration_test.go`. Fixed as proposed, except the F1 assertion noted below.
  - F1: a new row `--card carries a vet run that passed on the same clean tree` commits a tier1-tagged file, runs `--card 1` twice, and asserts the second call runs no tier invocation and prints three `carried` summary lines, the vet line among them.
    - Fixed differently: the fix proposed reusing the carry assertion, but `wantCarried` requires the `all` scope form, which a `--card` call never prints (`card carried`, `vet carried`).
      The row counts the `carried` lines and checks the vet line instead.
  - F2: the failing-vet row now asserts the vet run record's `TierList` is `[0]`, and that the guard step's run record exists after the failed vet.

`lyx gate test --tier 1 ./internal/gatecli` is green after the changes.

## Self-check

All four forks reported; the lens reports are saved beside this file.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
