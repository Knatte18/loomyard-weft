---
round: 1
focus:
  - "internal/raddleparser/check.go and proposal.go: does the check refuse a part with no module label and a part label outside the board vocabulary, while accepting a part label not among the Sketch's labels?"
  - "internal/loomshed/raddlefile.go: does the `labels` closure write each part's own module labels with its type label, and no longer the Sketch's whole set; are duplicates and ordering handled?"
  - "internal/loomcli/raddlecontext.go and internal/boardengine/raddlefile.go: does the vocabulary the check needs reach the parser, and do failure paths (empty or missing vocabulary) behave correctly?"
  - "contracts/specs/raddle-proposal-spec.md, raddle-template-agent.md and loom-rubric-raddle-review.md: does the stencil state the per-part label rule beside the other part fields, and does the split perch weigh each part's labels against the packages its body names?"
  - "Tests in raddleparser, loomshed and loomcli: do they cover the three check outcomes (missing, outside vocabulary, outside Sketch but inside vocabulary) and the filed labels, without duplicating each other?"
  - "Commit messages in the range: any Pattern-Departure trailer, and whether the diff stays within the Goal's Idea section."
---

Seed focus drawn from the board entry under `## Goal`, the only populated section of the decision record.
