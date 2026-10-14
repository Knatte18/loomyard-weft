---
round: 1
focus:
  - "internal/loomengine/prompt.go, discussion.go, contracts/stencils/loom/loom-template-discussion.md, contracts/stencils/stencils.go: the parent dropped the twill write-scope marker and the ready-marker wait while the task added a specs-directory marker to the Discussion stencil; settle whether composePrompt's values and optional lists, the stencil's markers and the claims/markers tests still agree, with no marker filled but absent or declared but unfilled."
  - "internal/loomengine/discussion.go and discussiontable.go with their tests: both sides edited the Discussion spec and seat table; settle whether the task's spec pointer and the parent's fork-after-record wording still describe one procedure in the twill path, so the forks review a record whose format the spec defines."
  - "internal/shedrecipe/recipe.go, internal/testkit/envkit/envkit.go, internal/loomcli/wiring.go and wiring_test.go: the parent removed the PrepareDiscussionSpawn and PreparePlanSpawn Env fields while the task added Env wiring for darn and the discussion gate; settle whether the task's wiring, fixtures and tests assume anything the parent removed, and whether any test the task added now duplicates one the parent kept."
  - "contracts/specs/refusal-spec.md and cmd/lyx/testdata/test-budget.yaml: both sides edited them; settle whether the task's rows or counts survived the merge in the form each side meant, and whether the budget rows match the packages' top-level test counts after both sides' test additions and deletions."
  - "pattern/PATTERN-cli-cobra.md and docs/overview.md: settle whether the task's new CLI behavior or module table entries follow the parent's changed deviations list and the overview's current module table."
---

Seed focus from the overlapping files and the parent's removals of the ready-marker and twill write-scope code.
