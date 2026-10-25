---
round: 1
focus:
  - "internal/planglyph/drift.go: do reported renames pass the same gates as the exact tier (declared Rename card pair skipped, old glyph no pending card references logged only, pair the exact tier already renamed skipped), and is the old glyph excluded from the deleted-symbol sweep and the evidence-tier candidates? Does a new glyph that fails to resolve yield a blocking finding?"
  - "internal/websterengine/recordbatch.go and recoverbatch.go: do both terminal verbs feed the same renames to drift detection through postBatchChecks, and does the digest carry `renames` additively so an older state reads as none?"
  - "internal/websterengine/report.go: does ParseReport refuse an empty side and old equal to new, with the recover-batch way-forward wording, and still accept and ignore an older `deviations` key?"
  - "internal/planparser/renumber.go: is the card-id rewrite bounded on both sides, padding-preserving, computed from the pre-renumber numbering in one pass, limited to moved cards, and journaled so a failed renumber restores every changed file? Is a rewritten id skipped by the prose-mention scan?"
  - "internal/loomcli/renumber.go: does the envelope report `rewritten_refs`, and do the verbs' --help and the docs (websterengine, planglyph, loomcli doc.go, loom-plan-spec numbering section) match the behavior?"
  - "Webster implementer and master stencils: do the changes state the plan-names-are-the-contract rule and the `renames` report field with its reason, without dropping or weakening a rule the answer key did not ask to change?"
  - "Tests: does any new test duplicate an existing one's coverage, and is any acceptance criterion's behavior (recover-batch path, failed-renumber restore, no bare-number re-report) left without coverage?"
---

Seed focus drawn from the board entry and the decision record's scope and acceptance criteria against the diff's file list.
