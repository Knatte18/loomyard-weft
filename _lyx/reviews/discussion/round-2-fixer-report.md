# Fixer report — discussion round 2

Every finding's premise held against the decision record and, for F6, against `internal/reedengine/state.go` (the persisted `Strand` carries `paneId` only) and `internal/reedengine/overlay.go` (`paneListFormat` reads `#{pane_pid}` live).

## Changes

- F1 (fixed as proposed): D4 refuses a peer in no known strand pane when the worktree list, a reed state or a pane process cannot be read, never resolving it as EXTERNAL or a program sender; the Linux identity criterion gains the case.
- F2 (fixed as proposed): D12 gives `post --wait-reply` and `inject --wait` a `--timeout` that exits non-zero, and says they wait until an outcome without one; D15 answers a `--wait-reply` whose message returns undelivered, exiting non-zero; D12's reconnect answer covers that outcome; D9's `--wait` points at D12's `--timeout`; Scope lists both `--timeout` flags; the reply criterion gains both cases.
- F3 (fixed as proposed, second option): D9 says a hold does not stop an inject's TTL, so another sender's hold can expire it; the inject TTL criterion gains the case; the support log records the pick (Q17) and the rejected alternative.
- F4 (fixed as proposed): D16 accepts and records `hold` and `release` on an orch, typing nothing; the orch criterion gains the case; the support log records the pick (Q18) and the rejected alternative.
- F5 (fixed as proposed): D4's off-Linux Bound names program-sender claims by any process speaking the socket protocol; the Open risks line says the same.
- F6 (fixed as proposed): D1 takes liveness and pane process from a read-only query of the hub's tmux server through reed.
- F7 (fixed as proposed): D4 accepts a program-sender name only under this hub's shortname; the Linux identity criterion gains the refusal.
- F8 (fixed as proposed): D9 and D12 use the D6 event name `injected` for the typed outcome; D16 and the support log call orch reminders withheld; D10, D13 and their criteria call a hold's sender the hold owner; D9 drops "refused" from the `--wait` outcomes.

## Disputed

None.

## Deferred

None.
