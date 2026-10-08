---
round: 3
ledger:
  - key: d10-pair-can-write-hubwide-landing
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-outside-hub-unslotted-unbounded
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: cli-wait-sec-vs-bash-timeout-chain
    rounds: [1, 2, 3]
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
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: acceptance-criteria-miss-decision-behavior
    rounds: [1]
    status: resolved
  - key: d7-cliwire-preflight-import-barred
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: card-verify-and-verify-fix-miss-background-rule
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-orch-prepush-step-absent
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 3 carries seven findings, all `design`, four of them BLOCKING.

Four sit on keys from earlier rounds.
`d10-pair-can-write-hubwide-landing` (F3) is open for the third round: the env-clearing route to the prime is unbounded, and the Goal still claims the Bound names every route.
`d7-outside-hub-unslotted-unbounded` (F2) is open for the third round: resolving the `-C` dir refuses a nested module.
`cli-wait-sec-vs-bash-timeout-chain` (F6) is open for the third round: the separate process group and the Windows mechanism leave the kill guarantee partial.
`failure-record-tamper-bound-unstated` (F4) rose from LOW consistency to BLOCKING design: D12 renders record fields into a command and a prompt.

New in round 3: F1 (D7 puts `preflight.ResolveMode` in `internal/cliwire`, which bars it), F5 (card `verify:` lines and verify-fix runs miss the background and slot-busy rules), F7 (the orch pre-push step does not exist).

Resolved since round 1 and not reopened: the `verifyrun` constraint wording, the slot-busy carve-out in D2 and the acceptance-criteria coverage.
