---
round: 1
ledger:
  - key: discussion-commit-pathspec-empty-parent-review-dir
    rounds: [1]
    status: resolved
  - key: loom-parent-arm-runs-for-review-verbs
    rounds: [1]
    status: resolved
  - key: wire-doc-comment-displaced-and-gofmt
    rounds: [1]
    status: resolved
  - key: verdict-expiry-check-then-act-race
    rounds: [1]
    status: resolved
  - key: fixed-column-wrapped-comments
    rounds: [1]
    status: resolved
  - key: hasround-doc-comment-misnamed
    rounds: [1]
    status: resolved
---

Round 1 reviewed the fourteen card commits over merge base `218ba2a68`.
It raised six findings: two BLOCKING, one MEDIUM, two LOW and one NIT.
The BLOCKING ones were the discussion commit failing on an empty or absent parent-review directory, and the loom parent's arm running for the review verbs under `cobra.EnableTraverseRunHooks`.
The MEDIUM was `wire`'s displaced doc comment together with the gofmt failure in wiring.go.
The LOW ones were a check-then-act race between `RecordVerdict` and `MarkExpired`, and fixed-column-wrapped comments.
The NIT was `hasRound`'s doc comment naming the wrong function.
The fixer resolved all six in one commit each, and the Bouncer verified each fix plus the full `verify:` block on HEAD `0aaccbfc9`.
No finding stays open.
