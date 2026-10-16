# Fixer report — discussion round 2

## Changes

- F1 (fixed as proposed): D6 gains a bullet saying the sweep that ends a pending confirm sets its next wait as if the previous wait were the base, so the backoff resumes from 30 s; a changed set or a new first miss follows the other rules.
  Premise checked: `sweepLoop` in `internal/creelengine/sweep.go` doubles from the current `wait` with `min(wait*2, sweepBackoffCeiling)`.
- F2 (fixed as proposed): Acceptance criteria gain a line that a failed listing after a first miss sets the next sweep at the confirm interval, and the holder is released at the first later successful listing that misses it again.
- F3 (fixed as proposed): D9's Bound now says claude shows the bypass gate to a strand whose launch flags or Claude settings ask for bypass permissions.
- F4 (fixed as proposed, test option): D9's Bound now says a seam-enforcement test in `internal/shuttleengine` pins that no production package outside `shuttleengine` calls the gate seam, and the PATTERN-shuttle-provider-seam constraint names that scan.
  The existing `seam_enforcement_test.go` scan over production files is the natural home.
- F5 (fixed differently in wording only): D4's Bound now says a start that fails before it tries the lock ends the wait even if another client's daemon is starting, and that the verb returns the daemon-log error unless its one last connect reaches that daemon.
  The proposed text said the verb returns the daemon-log error outright, but D4's Decision makes one last connect first, which can succeed against another client's daemon; the added clause keeps the Bound consistent with that.

## Disputed

None.

## Deferred

None.
