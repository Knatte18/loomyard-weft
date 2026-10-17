---
round: 2
ledger:
  - key: fan-setting-null-recording-undefined
    rounds: [1]
    status: resolved
  - key: darn-review-entry-model-sources
    rounds: [1]
    status: resolved
  - key: status-and-build-budget-shared-source
    rounds: [1]
    status: resolved
  - key: null-fan-contradicts-shape-refusal
    rounds: [1]
    status: resolved
  - key: envkit-setting-fake-left-open
    rounds: [1]
    status: resolved
  - key: pattern-resolved-value-home-outside-review
    rounds: [1]
    status: resolved
  - key: snapshot-rationale-parity-claim
    rounds: [1]
    status: resolved
  - key: negative-budget-refusal-key-name
    rounds: [1]
    status: resolved
  - key: constraints-bullet-restates-criteria
    rounds: [1]
    status: resolved
  - key: fan-setting-presence-agreement-unpinned
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: darn-routing-way-forward-names-wrong-table
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: caps-records-no-file
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's nine findings are resolved: none recurs in round 2, whose review confirms the null-fan, budget-parity and default-config paths as consistent.
Round 2 raised three new findings.
F1 (`fan-setting-presence-agreement-unpinned`, MEDIUM design) says D7's agreement test pins only the shared reference, so a fix row or Bouncer can omit its fan setting and bypass the recorded-round guard; it changes a decision and needs fresh review.
F2 (`darn-routing-way-forward-names-wrong-table`, LOW) and F3 (`caps-records-no-file`, NIT) are local wording corrections in D8.
