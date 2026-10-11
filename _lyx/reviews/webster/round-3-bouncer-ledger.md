---
round: 3
ledger:
  - key: retry-landed-slash-command-dropped
    rounds: [1]
    status: resolved
  - key: paste-buffer-leaks-on-failed-paste
    rounds: [1]
    status: resolved
  - key: clihelp-parallel-subtests-share-cobra-tree
    rounds: [2]
    status: resolved
---

Round 1 raised two LOW design findings in `internal/shuttleengine/send.go` and `internal/reedengine/io.go`; round 2 did not re-raise them.
Round 2 raised a BLOCKING design finding: parallel subtests of `TestRenderIndex_FormsAndFiltering` shared one cobra tree and failed the Publish verify with concurrent map writes.
Commit `321e93b13` made those subtests serial, and round 3 found nothing, so every key is resolved.
