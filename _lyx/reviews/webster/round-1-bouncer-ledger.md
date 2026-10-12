---
round: 1
ledger:
  - key: closeAlignment-nil-aligner-running-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: template-parent-review-wait-min-comment-spacing
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised two findings.
`closeAlignment` in `internal/parentreview/gate.go` calls `Aligner.Close` without a nil check, so a gate rebuilt with alignment off panics over a round whose `alignment.json` is running.
The `parent_review_wait_min` line in `internal/loomengine/template.yaml` lost one of its two spaces before the comment marker.
Both are open until a later round's fresh review shows them gone.
