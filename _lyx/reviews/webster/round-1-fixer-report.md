# Fixer-report — webster round 1

## Changes

- F1, fixed as proposed (commit `43d444292`):
  - `internal/reedengine/bar.go`: dropped the second `#[nolist align=right]%H:%M ` from `statusFormatSessions`, so the time segment appears once as at the merge base.
  - `internal/reedengine/bar_integration_test.go`: `assertSessionButtonsOnce` in `TestBar_LineTwoListsSessionsPrimeFirst` now asserts the expanded line 2 holds `nolist align=right]` exactly once, in all three prime-mark states.

`lyx gate test ./internal/reedengine` passed.
The new assertion sits under the `tmux` tag, so this round did not run it.

## Self-check

Four forks returned a report, none lost.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
