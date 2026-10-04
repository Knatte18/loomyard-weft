---
round: 1
ledger:
  - key: inbox-comments-wrapped-mid-sentence
    rounds: [1]
    status: open
  - key: selfreport-comments-fixed-column-wrapping
    rounds: [1]
    status: open
  - key: load-leaves-labels-nil
    rounds: [1]
    status: open
---

Round 1 raised three findings.
F1 (`inbox.go`) and F2 (`selfreport.go`) are BLOCKING comment-line-break violations.
F3 is a LOW design finding: `Load` in `internal/boardengine/store.go` leaves `labels` nil for entries stored with a kind and no labels key.
All three stay open until a fresh round reviews the fixed text.
