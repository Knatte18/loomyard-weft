---
round: 1
ledger:
  - key: comment-hard-wrap-multi-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: watcher-stopping-erased-by-tick-save
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-reload-stencil-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings, all open.
The BLOCKING one is hard-wrapped, multi-sentence Go comments in the diff, against `PATTERN-comment-line-breaks` (F1).
The gating design one is the watcher tick's whole-state save overwriting the persisted `WatcherStopping` record (F2).
The LOW one is the orch reload-pointer and role stencil comments describing a skills reload that a compaction no longer performs (F3).
