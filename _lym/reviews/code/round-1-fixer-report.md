# Fixer-report, code round 1

## Changes

- **F1** (LOW), fixed as proposed, commit `4359c542`: added the row `PrefixLongerThanForm` (`go build ./internal/x extra`) to `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` in `internal/shellguard/rows_test.go`. Removing the length guard in `refusedRaw` makes the row panic.
- **F2** (NIT), fixed as proposed, commit `3fc4b058`: the shuttle paragraph of `docs/overview.md` now says the raw go deny steers "to `lym gate test`, and to a package-scoped compile check the hub's `refuse_raw` does not refuse".

`lym gate test ./internal/shellguard` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
