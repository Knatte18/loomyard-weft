---
round: 2
ledger:
  - key: inject-unbounded-interrupt-key-path
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: still-busy-reading-vs-lagging-state
    rounds: [1]
    status: resolved
  - key: no-input-box-reason-misquoted
    rounds: [1]
    status: resolved
  - key: orch-retry-defeated-by-entry-short-circuit
    rounds: [1]
    status: resolved
  - key: orch-guard-refusals-silent
    rounds: [1]
    status: resolved
  - key: d1-overstates-no-drift
    rounds: [1]
    status: resolved
  - key: parent-picker-decision-contradicts-record
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: picker-close-esc-no-mechanism-or-bound
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: resume-busy-precheck-refuses-no-running-turn
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-needle-whole-capture-false-positive
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised six findings; round 2 resolves five of them and reopens one.
`inject-unbounded-interrupt-key-path` returns as F3: D7 now refuses Inject, but only for an input list equal to the interrupt sequence, so a double Esc still passes.
Round 2's two BLOCKING findings, F1 and F2, both stem from the decision the parent added after Discussion, which makes lyx detect and close an open rewind picker.
It contradicts Scope Out, D6, D7 and the support log, and it names no detection seam, sending path or bound.
F4 (resume's busy pre-check refusing a `no_running_turn` session) and F5 (D3's whole-capture needle) are new.
