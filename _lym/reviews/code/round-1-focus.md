---
round: 1
focus:
  - "internal/runprofile/template.yaml: does every darn block, `speed` and `docs` included, set `code_reviewer_model` to a valid Opus model-spec, and do the tapestry, pawl, sampler, motif and arras blocks stay unchanged?"
  - "Seed-only reconcile (internal/configsync, internal/hubreconcile, the profiles.template-base copy): does a hub with an unedited earlier profiles file pick up the new value while an operator's own `code_reviewer_model` is kept, and do the changed test fixtures pin that?"
  - "internal/runprofile/runprofile_test.go TestTemplateLadder: the old sampler-equals-darn `loom` assertion was replaced by a rounds-only comparison; does the new check still cover every key the sampler block must carry from darn, apart from the model?"
  - "Sampler runs: does the sampler block rightly keep the hub's Code-Review model, or does the entry's work-item review need it there too? Settle it against the board entry under `## Goal`, not the writer's claim."
  - "Test economy: TestTemplateDocsProfile and TestTemplateLadder are listed as contained in the redundancy report; compare their new `code_reviewer_model` assertions against TestSettings, TestLoad and TestFormerRecipeNameRefusal before filing."
  - "Commit message and doc.go/template.yaml comments against the diff: do the claims about which blocks take the hub's model match the template as left?"
---

The change is small: one template, its doc comments, and tests across five packages.
The first two entries carry the correctness risk; the rest check scope and test economy.
