---
round: 1
ledger:
  - key: orchcli-refusal-weft-vocabulary-leak
    rounds: [1]
    status: resolved
  - key: shuttleengine-claudeengine-comment-midsentence-breaks
    rounds: [1]
    status: resolved
  - key: orchengine-comment-midsentence-breaks
    rounds: [1]
    status: resolved
  - key: orchcli-comment-midsentence-breaks
    rounds: [1]
    status: resolved
  - key: smoke-background-turn-wait-not-implemented
    rounds: [1]
    status: resolved
  - key: smoke-open-risk-observations-never-gathered
    rounds: [1]
    status: resolved
  - key: orchengine-doc-clearing-timeout-misstated
    rounds: [1]
    status: resolved
  - key: watcher-abort-returns-cursor-untested
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings against the committed diff: one BLOCKING, five MEDIUM and two LOW.
The BLOCKING one was a bare `weft` token in `orchcli/refusal.go` that turned `internal/lyxcwd`'s vocabulary test red.
The MEDIUM ones were fixed-column comment wraps across shuttleengine, claudeengine, orchengine and orchcli, and two smoke-test gaps: a background-task wait that could pass before the turn began, and open-risk observations that could never gather answers.
The LOW ones were a misstated clearing timeout in `orchengine/doc.go` and a missing test pinning the cursor on three abort returns to idle.

The fixer landed one commit per finding.
The judge re-verified each at HEAD: the full test suite, the integration set and the smoke vet all pass, and the comment scan finds no mid-sentence breaks.
All eight are resolved, and nothing is carried open.
