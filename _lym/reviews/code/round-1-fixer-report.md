# Fixer-report, code round 1

## Changes

- F1: fixed as proposed (commit `dd5941ae`). The passing `--card` vet row in `internal/gatecli/scopes_integration_test.go` now asserts the not-run line naming tier1 among its tiers, which fails if the `vetStep` exclusion in `notRunTiers` is dropped.
- F2: fixed as proposed (commit `b156d413`). `printNotRunTiers` in `internal/gatecli/run.go` builds the line from `tierNames(numbers)`.
- F3: fixed differently. The review's `fix` key offers two changes and names accepting the shorter wording as the smaller one; I took it, so the code and tests are unchanged. `doc.go:10` already states the shorter line, and no change was owed.

`lym gate test --upto 2 ./internal/gatecli` passed after F1 and F2.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
