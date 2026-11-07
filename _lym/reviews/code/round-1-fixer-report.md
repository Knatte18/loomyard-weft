# Round 1 fixer-report

## Changes

All edits are in `_lym/landing/summary.md`. Both findings' premises held (grep: five `beamHub` call sites, fifteen `newWiredPairFixture` call sites).

- F1, fixed as proposed: the `newWiredPairFixture` bullet no longer claims two sites and a three-site threshold.
  It now says the task names only `beamHub` and `newBadReedUpFixture`, so the fixture is left out of this change.
- F2, fixed as proposed: the `beamHub` count is dropped ("several tests and subtests").
  The same stale tally in "To check by hand" and the `newBadReedUpFixture` count, which was true, are dropped too, so no site tally remains in the prose.

## Disputed

None.

## Deferred

None.

Git: no commit was made. The only file changed is under `_lym`, and this round forbids running git against a `_lym` path.

## Self-check

All four forks returned a report:

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
