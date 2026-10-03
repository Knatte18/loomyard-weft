---
round: 2
ledger:
  - key: seam-import-allowlists-lack-stale-detection
    rounds: [1]
    status: resolved
  - key: cli-walk-bogus-accepts-any-refusal
    rounds: [1]
    status: resolved
  - key: cli-tree-findings-never-notice-fix
    rounds: [1]
    status: resolved
  - key: dropped-concept-claims-case-sensitive
    rounds: [1]
    status: resolved
  - key: tmux-isolation-per-file-platform
    rounds: [1]
    status: resolved
  - key: tmux-isolation-kit-call-by-identifier
    rounds: [1]
    status: resolved
  - key: changed-comments-fixed-column-wrapped
    rounds: [1, 2]
    status: resolved
  - key: card29-helpers-detached-doc-comments
    rounds: [1]
    status: resolved
  - key: leaf-test-counts-unrecorded
    rounds: [1, 2]
    status: resolved
  - key: step-envelope-docs-restate-names
    rounds: [1]
    status: resolved
  - key: sectionof-trailing-clause-untested
    rounds: [1]
    status: resolved
---

Round 1 reviewed the whole branch over its merge base with `main` (cards 1–34), with no rework round, and its fixer resolved nine of ten items plus a self-found detached-doc-comment defect from card 29.

Round 2 carried the two items left open.

- **leaf-test-counts-unrecorded:** resolved by `c317fa5a6`, a body-only commit recording leaf counts for both suites before card 1 and after the last card, with per-package drops tied to the card commits that name each deletion; the itemisation sums to the measured net (−66 untagged, −67 integration).
- **changed-comments-fixed-column-wrapped:** resolved by `ebd7e2e4c` (`cmd/lyx/seamsignature_test.go`, now broken only at its independent "and" clause) and `3f43c4e21` (`internal/loomrecipe/shape_test.go`, now one sentence per line).

No item is open after round 2.
