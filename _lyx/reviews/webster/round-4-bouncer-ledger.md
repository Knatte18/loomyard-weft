---
round: 4
ledger:
  - key: hard-wrapped-new-comments
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: BLOCKING
  - key: held-run-tests-not-parallel
    rounds: [1]
    status: resolved
  - key: stale-asking-outcome-prose
    rounds: [1]
    status: resolved
  - key: session-continued-in-corpus-duplicate
    rounds: [1]
    status: resolved
  - key: send-tail-untested
    rounds: [2]
    status: resolved
  - key: dead-finalize-message-plumbing
    rounds: [2]
    status: resolved
  - key: diedmaster-call-sites-not-gofmt
    rounds: [2]
    status: resolved
---

Round 4 maps F1 to F5 to `hard-wrapped-new-comments`.
The key has been open in every round: each round fixed the sites the review named, and the next fresh review found more sentences whose rewritten tail still breaks mid-phrase.
Round 4's sites are `attach.go`, `attach_test.go`, `treadleengine/engine_test.go`, `shuttlecli/smoke_guardrail_test.go` and `claudeengine/settings.go`.
The class stays `consistency` as in the review, since no finding entry shows a mislabel.
The six other keys stay resolved and were not raised again.
