---
round: 1
ledger:
  - key: gitwrap-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: docgo-merge-only-paragraph-wrap-and-structure
    rounds: [1]
    status: resolved
  - key: record-recover-batch-edited-comments-wrap
    rounds: [1]
    status: resolved
  - key: runinflight-doc-line-breaks
    rounds: [1]
    status: resolved
  - key: merge-verbs-comment-line-breaks
    rounds: [1]
    status: resolved
  - key: merge-in-warning-overclaims-fast-forward
    rounds: [1]
    status: resolved
  - key: new-test-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: merge-in-tests-do-not-pin-warning-content
    rounds: [1]
    status: resolved
  - key: reconcile-tests-underassert-card1
    rounds: [1]
    status: resolved
  - key: mid-merge-refusal-prefix-mismatch
    rounds: [1]
    status: resolved
  - key: redundant-newhead-alias
    rounds: [1]
    status: resolved
---

Round 1 raised eleven findings, F1–F11.
Six are LOW: five concern comment line breaks and one concerns the merge-in warning over-claiming for a fast-forward.
Five are NIT: test fidelity, the refusal prefix, and a redundant alias.
The fixer landed one commit per finding (`2b5cdb05e`..`a3028506c`), and each fix is present in the committed diff, so every entry is resolved.
No finding remains open.
