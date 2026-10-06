---
round: 2
ledger:
  - key: held-compaction-boundary-survives-stale-tick
    rounds: [1]
    status: resolved
  - key: plugins-step-not-carved-out-of-resuming-tick
    rounds: [1]
    status: resolved
  - key: loom-start-smoke-tests-assert-handover
    rounds: [1]
    status: resolved
  - key: pattern-cli-cobra-lists-orch-start
    rounds: [1]
    status: resolved
  - key: attach-probe-only-callers-inherit-teardown
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: running-turn-case-has-no-test-row
    rounds: [1]
    status: resolved
  - key: cards-1-to-7-name-no-module-doc
    rounds: [1]
    status: resolved
  - key: card-8-omits-arm-go
    rounds: [1]
    status: resolved
  - key: card-16-pins-wait-twice
    rounds: [1]
    status: resolved
  - key: card-18-tests-verb-twice
    rounds: [1]
    status: resolved
  - key: submit-redraw-settle-left-open
    rounds: [1]
    status: resolved
  - key: plugins-step-advance-underspecified
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: held-boundary-survives-strand-change
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: missing-stop-message-passes-read-turn-end
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eleven findings, two of them gating (the held compaction boundary and the plugins step in the resuming tick).
Round 2 no longer raises either of those; the other round-1 findings are resolved except the attach-probe teardown, which round 2 reopens as F1 with the same cause: the teardown still reaches callers that start no run.
Round 2 adds three LOW findings: the plugins step advance is underspecified (F2, `design`), a held boundary survives a strand change (F3, `scope`), and a missing Stop message passes the read-turn-end check (F4, `design`).
None is gating.
