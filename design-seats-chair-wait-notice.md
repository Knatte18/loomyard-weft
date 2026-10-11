# Seats chair planned waits reach the parent as stall notices

`seats-chair-wait-notice` · Note · bug, shuttle, loom

## Seen (2026-10-11, discussion bench)

Every time a seats chair ends its turn to wait, for an advisor's up line or for an advisor's answers, shuttle sends the parent a "turn end held, output files missing" notice (`internal/shuttleengine/hold.go:53`, `notifyHeld`).
These waits are planned by the chair stencil, so the notices are false stall alarms; each advisor run in the bench sent the orch three to four of them.
Advisors are quiet because `internal/seatengine/prompt.go:160` sets `QuietHold: !isChair`; the chair is not.
Answering a notice wakes the chair for a turn, which sends another notice.

## Idea

Suppress the held notice for a chair while any advisor has an unanswered batch or has not yet sent its up line, and keep it for a chair that ends a turn with no advisor work outstanding, which is a real stall.
