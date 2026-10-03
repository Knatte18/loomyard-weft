---
round: 2
ledger:
  - key: tier1-under-step-unlicensed
    rounds: [1, 2]
    status: resolved
  - key: anomaly-pending-retry-unlicensed
    rounds: [1, 2]
    status: resolved
  - key: failed-halt-envelope-drops-friction
    rounds: [1]
    status: resolved
  - key: halt-filing-intent-unbounded
    rounds: [1]
    status: resolved
  - key: reflect-settle-then-spawn-double-timeout
    rounds: [1]
    status: resolved
  - key: reflect-record-failures-unspecified
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-untested
    rounds: [1]
    status: resolved
  - key: drive-stop-report-friction-unlicensed
    rounds: [1]
    status: resolved
  - key: step-hook-impactsummary-misnames-consumers
    rounds: [1, 2]
    status: resolved
  - key: armedverb-removal-misses-wiring-test
    rounds: [2]
    status: open
  - key: should-reflect-friction-removal-misses-approve-awaiting-test
    rounds: [2]
    status: open
---

This is generation 0: there is no `_lyx/loom/rework/`, so the decision record is the whole answer key.

Round 1 left two scope findings open.
Cards ran Tier 1 anomaly filing under `step` and added a pending-retry marker, and the decision record puts both out of scope.
In round 2 the fixer deleted both cards and every part of the remaining cards that depended on them.
The plan now has nine cards.
The overview gained a Shared Decision recording that the board's post-Discussion additions are not carried and need Discussion first.
Card 2's ImpactSummary now names `battencli` and `shedcli`.
Every round-1 finding is resolved.

Round 2 found two new footprint defects, both in `internal/loomcli` tests and both BLOCKING.
Card 3 removes `loomCLI.armedVerb`, but `TestArmAt_RecordsTheArmingVerb` in `wiring_test.go` still reads it.
Card 6 removes `shouldReflectFriction`, but `TestShouldReflectFriction_AwaitingNeverReflects` in `approve_awaiting_test.go` still calls it.
Neither test file is in its card's targets, and each card says the symbol has no other reader or caller.
