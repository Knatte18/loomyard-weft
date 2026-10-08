---
round: 2
ledger:
  - key: d10-pair-can-write-hubwide-landing
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-outside-hub-unslotted-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: cli-wait-sec-vs-bash-timeout-chain
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: verifyrun-constraint-wrong-pattern-open-seam
    rounds: [1]
    status: resolved
  - key: d2-failure-rule-no-slot-busy-carveout
    rounds: [1]
    status: resolved
  - key: failure-record-tamper-bound-unstated
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: acceptance-criteria-miss-decision-behavior
    rounds: [1]
    status: resolved
---

Round 2 carries four findings, each on a key from round 1.

The BLOCKING one (F1) is still `d10-pair-can-write-hubwide-landing`: D10 keys its refusal on the resolved worktree, so a pair passes it from the prime's directory, and the Goal claims otherwise.
F2 reopens `d7-outside-hub-unslotted-unbounded` on a new ground: the `-C`-based hub resolution D7 relies on does not exist in `cliwire`.
F3 reopens `cli-wait-sec-vs-bash-timeout-chain`: the wait is tied to the Bash timeout, but the run itself, and whether `go test` dies with a killed holder, is unbounded.
F4 narrows `failure-record-tamper-bound-unstated`: an edited record also widens, not only skips or narrows.

Resolved with no round 2 finding: the `verifyrun` constraint wording, the slot-busy carve-out in D2 and the acceptance-criteria coverage.
