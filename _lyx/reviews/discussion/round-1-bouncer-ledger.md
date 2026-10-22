---
round: 1
ledger:
  - key: way-forward-command-cannot-clear-dirt
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: handover-restep-bypasses-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: routed-from-premise-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-record-admit-yields-discard-text
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: refusal-arm-no-spec-class
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: false-reason-for-goto
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: path-list-collapses-to-one-line
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ac1-contradicts-cap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-premise-lacks-evidence
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: settled-fact-filed-as-assumption
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: note-points-at-deleted-function
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: deliberation-in-scope
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings.
The BLOCKING one is that the way forward cites a command that cannot clear the parent checkout's tracked dirt.
Two further design MEDIUMs concern the driver's hand-over route re-stepping a blocked Finalize, and a false premise behind the second guard target kind.
The rest are consistency, decision-record placement and evidence gaps, mostly mechanical.
