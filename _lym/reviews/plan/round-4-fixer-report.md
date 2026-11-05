# Round 4 fixer-report — plan

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  `_lym/plan/22-driftcli-retire-verbs.md`: removed the `internal/driftcli#compareLockRetry` bullet under `**Delete:**`; nothing else in the card changed.
  The constant stays live through the compare closure of `worktree.checkDeps` (`internal/driftcli/wiring.go:247`), which no card removes.
- **Gate finding `caller-uncovered/25-driftengine-retire-acceptance` (blocking), named in the review's Rejected section as one the fixer meets at the gate.**
  `_lym/plan/25-driftengine-retire-acceptance.md`: removed the `internal/driftengine#testFiles` bullet under `**Delete:**`, and the `accepted.go` Intent bullet now says the file keeps `testFiles` beside `copyFile`, because `Check` uses it to leave a skipped test's cases out of an in-flight `partial` scope's diff.
  `testFiles` is read by `skippedCases` (`internal/driftengine/check.go:274`), which `Check` calls when `scope.Scope == ScopePartial` (`check.go:247`); no card removes `skippedCases`, and the card's `records.go` bullet keeps the `partial` scope readable.
  Its other readers, in `accepted.go` and `RunAcceptanceCapture` (`capture.go:302`), go with the functions card 25 deletes.

`lym loom validate-plan` reports `{"ok":true}` after both changes.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
