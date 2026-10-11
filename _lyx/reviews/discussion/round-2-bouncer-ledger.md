---
round: 2
ledger:
  - key: round-gate-has-no-merge-base
    rounds: [1]
    status: resolved
  - key: merged-in-parent-growth-blocks-branch
    rounds: [1]
    status: resolved
  - key: no-usable-base-pass-unbounded
    rounds: [1]
    status: resolved
  - key: proc-scope-vs-acceptance-line
    rounds: [1]
    status: resolved
  - key: round-gate-failure-no-way-forward
    rounds: [1]
    status: resolved
  - key: new-background-file-not-linked
    rounds: [1]
    status: resolved
  - key: gofmt-parse-failure-no-acceptance-line
    rounds: [1]
    status: resolved
  - key: trim-room-stated-two-ways
    rounds: [1]
    status: resolved
  - key: d7-perishable-gofmt-file-list
    rounds: [1]
    status: resolved
  - key: headroom-byte-count-overstates-cut
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised nine findings, centered on the round gate's base; round 2 no longer raises any of them.
Round 2 raised one new finding: when main sits above the line, D5's headroom finding names `head - line` bytes, but D2's pass rule is satisfied by cutting back to the base size.
D2 and D5 need to state one rule and an acceptance row for it.
