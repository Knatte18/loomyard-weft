---
round: 2
ledger:
  - key: twin-probe-until-done-helpers
    rounds: [1]
    status: resolved
  - key: no-trust-inputs-duplicates-undismissable-engine
    rounds: [1]
    status: resolved
  - key: card3-cli-row-repeats-engine-and-cli-rows
    rounds: [1]
    status: resolved
  - key: card6-intent-omits-what-shorter-guard-admits
    rounds: [1]
    status: resolved
  - key: card11-gate-poll-exit-conditions-unnamed
    rounds: [1]
    status: resolved
  - key: card5-apply-carried-unknown-of-unowned
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings; the round 2 review did not raise any of them again, so all five are resolved.
Round 2 raised one LOW consistency finding on card 5: `pool.applyCarried` is listed under Uses but has to report whether it folded a record, so that `pool.apply` adds a log entry only for a known `of`.
Its fix is a local edit the finding text fixes, so it is marked `mechanical`.
