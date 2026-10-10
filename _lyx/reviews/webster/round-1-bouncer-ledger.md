---
round: 1
ledger:
  - key: advisor-unstoppable-held-name-start-error
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: errorexit-cancelled-context-errseatnotstopped-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: seatengine-doc-reservedmarkers-reader-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings.
F1 (BLOCKING, design): in `Engine.Run`'s advisor loop, a `startSeat` error wrapping `ErrSeatNotStopped` is treated as an ordinary start failure, leaving a live strand with no handle.
F2 (LOW, scope): the `errorExit` branch returning `ErrSeatNotStopped` over a cancelled context has no test and departs from the plan wording.
F3 (LOW, consistency): `seatengine/doc.go` names a production reader of `ReservedMarkers` that does not exist.
