# Fixer-report, code round 1

## Changes

- **F1** fixed as proposed (commit `a3d6e3a2`).
  The failure branch of `TestStartup_LoadTurnAPIError` now fails a start whose virtual time reaches `wantFailedAfter + apiErrorRetryBackoff`, only when `wantFailedAfter > 0`.
- **F2** fixed as proposed.
  `_lym/landing/summary.md` now says each of the doubling, the ceiling and the busy session's back-off is pinned to within one `apiErrorRetryBackoff` above its scheduled wait.
  The file lies under `_lym`, so it is edited in the working tree and not committed.
- Self-check follow-ups, in one commit with the `wantFailedAfter` comment:
  - `startup_test.go` gained a comment on `wantFailedAfter` stating its upper bound.
  - `summary.md` now mentions the failing-row bound in the "Back-off from above" bullet and the open-turn-after-hold bullet.

Tier 0 gate on `./internal/shuttleengine`: 1309 passed, 0 failed.

## Self-check

- fix-consistency, LOW: `wantFailedAfter` had no comment on its new upper bound. Fixed as proposed.
- fix-consistency, LOW: `summary.md` bullets did not mention the failing-row bound. Fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
