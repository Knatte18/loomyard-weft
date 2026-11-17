# Relevant files

- `contracts/stencils/burler/burler-step-fix.md` — the fixer's dispositions, dispute grounds and fixer-report sections D1 turns into frontmatter
- `contracts/stencils/burler/burler-step-review.md` — the reviewer's prior-rounds marker D2 fills with the prior-findings block
- `contracts/stencils/burler/burler-review-format.md` — the review frontmatter D3 extends with `prior` and `carries`
- `contracts/stencils/burler/burler-template-fixer.md` — the fixer seat stencil naming the fix step and review format
- `contracts/stencils/burler/burler-template-reviewer.md` — the reviewer seat stencil
- `contracts/stencils/burler/burler-step-1-explore.md` — the reviewer's explore step, where the whole target is read today
- `internal/burlerengine/prompt.go` `priorRoundsBlock`, `composeFixPrompt`, `fixScopeRules` — renders the clean-room prior-rounds text D2 replaces and the fix scopes D5 reads
- `internal/burlerengine/profile.go` `FixScope`, `Profile` — the source and overlay fix scopes D5 snapshots
- `internal/shedadapters/escalation.go` `EscalationCause`, `writeEscalation`, `readEscalation` — the four causes D6 extends with `stalemate`
- `internal/shedadapters/bouncerfacts.go` `circlingEvidence` — Go's evidence guard on the judge's circling verdict
- `contracts/recipes/tapestry-recipe.yaml` — the Bouncer, review, fix ring and the per-perch budget keys
