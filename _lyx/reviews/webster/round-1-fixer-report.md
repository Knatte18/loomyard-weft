# Fixer-report, webster round 1

## Changed

- F1: `circlingVerb` (`internal/loomcli/circling.go`) now refuses at the hard cap only when the latest round has a pending escalation, read through a new `escalationPending` dep over `shedadapters.PendingEscalation`.
  A round with no escalation, or one already decided, falls through to `record`, which returns the existing not-escalated or already-recorded refusal.
  A malformed escalation record falls through too, so `record` gives the malformed-file way forward.
  The circling test table gained a not-escalated row and a decided row at the cap; the existing cap rows set `pending`.
  A legacy round escalated only by a CIRCLING verdict has no escalation record, so it is not pending and a `continue` at the cap is not refused; those runs predate hard caps.
- F2: `bouncer-fragment-decide-capped.md` now says the segment can only be accepted.
  The two tests that pinned the old phrase (`contracts/stencils/claims_test.go`, `internal/shedadapters/bouncer_escalation_test.go`) now pin "can only be accepted:".

Gate: `lyx gate test` green over `./internal/loomcli`, `./internal/shedadapters`, `./contracts/stencils`, `./contracts/specs`.

## Disputed

None.

## Deferred

None.
