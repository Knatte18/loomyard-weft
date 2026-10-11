---
round: 1
ledger:
  - key: probe-round-reviewer-only-stop-undecided
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: gate-failed-reviewer-branch-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: resume-absent-table-refusal-unreached
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: burlercli-doc-update-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: fork-subagents-flag-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card8-deletes-last-caller-symbols
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: single-implementer-seat-interface
    rounds: [1]
    status: open
    class: design
    severity: NIT
---

Round 1 raised seven findings, none gating.
The largest is the `ProbeRound` stop behaviour in card 8 (F1), a consistency gap.
The rest are LOW reachability, bound and carrier gaps, plus one NIT on a one-implementer interface.
