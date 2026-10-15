---
round: 1
ledger:
  - key: recordless-send-interrupt-ignore-strand-engine
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised one finding.
Recordless `Send` and `Interrupt` in `internal/shuttleengine` resolve the default engine, though `Open` can start a session on another engine and stamps it as the strand's provider tag.
The fix changes an interface (`StrandStatus` exposing `Provider`) and the engine-resolution contract, so it is `design-wrong` and needs a fresh review.
