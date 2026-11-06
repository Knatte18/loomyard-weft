---
round: 1
focus:
  - "internal/discussionparser/append.go and patternchanges.go, internal/loomcli/decision.go: does the withdraw option strike exactly the named PATTERN-change bullet in the same commit as the decision, and what happens when the id is absent, duplicated, or declared under another action (add, change, remove)?"
  - "internal/loomcli/decision.go and decision_test.go: is every failure path of the new option (unknown id, bad flag combination, write failure after the strike) reported and leaving no half-applied record?"
  - "internal/raddleparser/proposal.go, check.go and internal/loomshed/raddlefile.go: when a Sketch id maps onto a live cluster, is the part record's PATTERN changes section written under the cluster ids, and are unmapped ids and multi-id mappings handled without regressing the existing parse and check rules?"
  - "internal/loomcli/raddlecontext.go: does the context handed to raddle carry what the mapped-cluster filing needs, and is it consistent with the raddleparser check?"
  - "Docs and contracts (orch-role.md, loom-discussion-spec.md, raddle-proposal-spec.md, raddle stencil, docs/overview.md, package doc.go files): do they state the same behavior the code implements, and does the repository's own docs-with-code rule hold for this kind of change?"
  - "Test economy: do the new tests in append_test.go, partsplice_test.go, decision_test.go, check_test.go, raddlefile_test.go overlap each other, and is any added behavior left uncovered?"
  - "Commit messages and summary: does the 2000-byte Long cap fix commit change only help text, and does any claim in the change description fail against the diff?"
---

The decision record holds only the board entry under `## Goal`: a `lym loom decision add` option that withdraws a declared PATTERN change, plus raddle writing mapped changes under the live cluster ids.
Focus entries follow the two halves of that entry, then docs and tests.
