---
round: 1
ledger:
  - key: tackler-bound-unenforced
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: verify-goto-record-match-and-retry-split
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: handover-undefined-guard-and-crash-window
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: typed-child-driver-dropped-on-adopted-seed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: empty-reason-paused-is-by-design
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: transient-row-departs-from-llm-driver
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: watched-read-pid-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: deliberation-in-decision-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: resume-long-help-scope
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings, none carried from an earlier round.
The gating ones concern missing bounds and false premises in the Go-driver design: the tackler's unenforced read-only bound, the weak verify-record match with a split retry count, the undefined clean-tree guard with a hand-over crash window, a silently dropped typed flag, and the empty-reason paused premise.
The rest are a consistency departure in the transient row, a pid-only watched read, relocatable deliberation, and an incomplete help-text note.
