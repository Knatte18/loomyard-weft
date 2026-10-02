---
round: 2
ledger:
  - key: strand-name-retirement-mechanism
    rounds: [1]
    status: resolved
  - key: session-name-provider-degradation
    rounds: [1]
    status: resolved
  - key: ly-hub-workflow-doc-home
    rounds: [1]
    status: resolved
  - key: worktree-parent-seed-self-alias
    rounds: [1]
    status: resolved
  - key: support-log-superseded-hub-position
    rounds: [1]
    status: resolved
  - key: criteria-bad-slug-refusal-and-package-docs
    rounds: [1]
    status: resolved
  - key: redundant-hand-started-seed-criterion
    rounds: [1]
    status: resolved
  - key: dated-deliberation-in-rationales
    rounds: [1]
    status: resolved
  - key: standalone-mode-naming-undecided
    rounds: [1, 2]
    status: resolved
---

Round 1 resolved eight findings: the `strand_name` retirement, the provider's degradation behaviour, the hand-started hub path, the default-run seed path, the support log's view of the hub review, two missing criteria, a redundant criterion and dated rationales.
Its judge left one finding open: the record never decided how strands are named in standalone mode.

Round 2's reviewer raised that same gap as F1 (BLOCKING), and the fixer applied the proposed fix.
A standalone code is `s` plus the first five hex digits of the hash8.
`agentname` owns that rule, and `standalonegeom.ReedGeometry` tells it to reed.
Standalone strands take the two-segment form, standalone never sets `LYX_PARENT`, and the `strand_name` retirement applies there unchanged.
A hub never takes a derived code.
The Constraints, Mirrors, Notes, acceptance criteria and support log all match.
Checked against the code: hash8 is lowercase hex, so the derived code fits the code grammar.

No finding remains open.
