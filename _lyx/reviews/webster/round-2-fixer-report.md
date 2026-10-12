# Fixer-report — webster round 2

## Changes

- F1 (fixed as proposed): `internal/lyxtemp/sweep.go` — `Sweep` now removes a dead run directory through the new `removeRunDirectory`, which tries `os.RemoveAll` up to `removeAttempts` (three) times and treats a path that `os.Lstat` reports as not-exist as removed.
  The `ownerDead` doc comment now says the probe may briefly recreate a lock file a concurrent sweeper has just removed.
  `TestSweep/two_concurrent_sweeps_both_succeed` is unchanged and `lyx gate test ./internal/lyxtemp` passes.
- Gate finding (overview headroom): the merge grew the `PATTERN.md` overview from 15675 to 15911 bytes, past the headroom line.
  Shortened the wording of the `PATTERN-parent-lock`, `PATTERN-config-layers`, `PATTERN-hub-wide-config`, `PATTERN-pane-text-injection`, `PATTERN-lyx-temp-root`, `PATTERN-agent-filed-issues` and `PATTERN-review-round` entries; the overview is now 15672 bytes, no larger than its base.
  `lyx gate test ./internal/pattern` passes.

## Disputed

None.

## Deferred

None.
