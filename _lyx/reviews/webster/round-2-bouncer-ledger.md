---
round: 2
ledger:
  - key: advisor-unstoppable-held-name-start-error
    rounds: [1]
    status: resolved
  - key: errorexit-cancelled-context-errseatnotstopped-untested
    rounds: [1]
    status: resolved
  - key: seatengine-doc-reservedmarkers-reader-claim
    rounds: [1]
    status: resolved
  - key: seatengine-refusals-missing-from-refusal-spec
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: shedadapters-doc-errseatnotstopped-archive-claim
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's three findings are resolved: round 2's focus verdicts found the advisor held-name path, the cancelled-context `errorExit` row and the `ReservedMarkers` doc sentence correct.
Round 2 raised two new findings.
F1 (MEDIUM, decision): the seat engine's held-name and not-stopped refusals have no row in `contracts/specs/refusal-spec.md` and no entry in its out-of-scope list.
F2 (LOW, consistency): `internal/shedadapters/doc.go` says an `ErrSeatNotStopped` error is returned without archiving, but one from `Run` follows the archive.
