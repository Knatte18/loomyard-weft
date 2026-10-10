---
round: 1
ledger:
  - key: socket-check-runs-after-sweep
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: reed-servers-unregistered-with-tmuxkit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: reed-exit-empty-probe-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
  - key: pattern-overview-cap-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: exit-empty-early-return-false-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: warplayout-test-not-read-only
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: prebuilt-variable-not-stripped-on-skip
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: package-docs-state-false-bounds
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: rev-parse-survivors-omits-mergeheadpresent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised nine findings against the plan, none carried from an earlier round.
The gating ones concern the tmuxkit design (socket check ordering, unregistered reed servers, the exit-empty bound), the unlicensed reed production change, the uncarried PATTERN overview cap, and the prebuilt-variable strip.
The rest are classification, package-doc and survivor-list consistency.
