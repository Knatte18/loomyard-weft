---
round: 1
ledger:
  - key: round-gate-has-no-merge-base
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: merged-in-parent-growth-blocks-branch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: no-usable-base-pass-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: proc-scope-vs-acceptance-line
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: round-gate-failure-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: new-background-file-not-linked
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: gofmt-parse-failure-no-acceptance-line
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: trim-room-stated-two-ways
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: d7-perishable-gofmt-file-list
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings against the decision record.
The core defect is the round gate's base: it diffs from the plan verify's last pass rather than a merge base, so card-phase growth goes unchecked and a merge-in after a Publish failure counts as the branch's growth.
The unbounded no-usable-base pass and the missing refusal way forward follow from the same gate wiring.
The remaining findings are consistency and wording fixes.
