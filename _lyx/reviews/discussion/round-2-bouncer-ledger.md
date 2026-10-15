---
round: 2
ledger:
  - key: d3-multiline-clear-undecided
    rounds: [1]
    status: resolved
  - key: d3-post-settle-restore-timing
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-bound-counts-caret-cells
    rounds: [1]
    status: resolved
  - key: d5-start-match-by-time
    rounds: [1]
    status: resolved
  - key: d2-progress-line-stream
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps-d2-d8
    rounds: [1]
    status: resolved
  - key: d3-multiline-clear-criterion-passes-single-cu
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d5-concurrent-starts-interleave-log
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-blanked-reads-omit-d3-box-reads
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, two gating at MEDIUM, both about D3's interrupt clear.
Round 2 no longer raises the undecided clear sequence, the caret-cell bound, start selection by time, the progress-line stream or the D2 to D8 criteria gaps, so those keys are resolved.
The restore-timing key reappears as a LOW wording defect: the window is the send's guard-read-to-paste interval, not bounded by the settles.
Two new MEDIUM design findings came from the round-1 fixes: the multi-line clear criterion passes on a one-`C-u` clear, and D5's pid header does not separate two concurrent starts.
One new LOW consistency finding: D1's list of blanked box reads omits D3's three reads.
