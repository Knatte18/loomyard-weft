---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/burler/burler-focus-directive.md:3
    summary: banner names composePrompt in prompt.go as the call site, a function card 10 deleted
  - id: F2
    severity: LOW
    class: consistency
    location: internal/burlercli/cli.go:58
    summary: comment cites burlerengine.Result, a type card 10 deleted
  - id: F3
    severity: LOW
    class: consistency
    location: contracts/stencils/seat/seat-directive-advisor.md:4
    summary: seat directive banners say the Burler seat stencils state a narrower channel, but no Burler seat stencil survives
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, ten commits, since no rework round exists and `product.parent` is `main`.
`lyx gate test` over `./cmd/lyx`, `./internal/shedadapters`, `./internal/burlerengine`, `./internal/burlercli`, `./internal/shedrecipe`, `./internal/loomrecipe`, `./internal/loomengine`, `./internal/loomcli`, `./contracts/stencils`, `./internal/testkit/shedfake` and `./internal/loomshed` passes, which includes the test-budget check.

No BLOCKING or MEDIUM defect found.
The focus questions are settled as follows.

- Card 10 deletions: a repository-wide grep for every deleted symbol, stencil, registry name, the four `Name*Burler` constants and the ready marker finds no code, test, recipe or testdata reference.
  `.gitattributes` and `claims_test.go` entries for the deleted stencils are gone too.
  The only survivors are the comments in F1 to F3.
- Acceptance record: `writeAcceptance` is called only from `BurlerReviewProducer.acceptedExit`, reached only for `OutcomeDone`, which `concludeReview` returns after `acceptReview` runs `ParseCurrentReview`.
  Every whole-round archive (`archiveWholeRound`, the review row's `archiveRound`) lists review, fixer report and acceptance record together.
  A changed review ends `BurlerFixProducer.errorExit` with the goto-to-review-row way forward, and a missing acceptance record refuses with the same way forward.
- Fix row spawn: `Engine.Fix` spawns on any finding, runs the gates once on a clean review and skips with a written report only when they pass; `lyx burler run` calls `Review` then `Fix` only for a done reviewer, so it matches the recipes.
- Auto-accept and keys: `applyReviewBudget` is the one place where 0 and absent fall back to `review_max_bounces`, keyed by each segment's Bouncer row name; `New` and `Routing` both call it, and `bouncerEntry` reads `RowAutoAccept[name]`.
  The Darn recipe passes a nil map and keeps `darn.yaml`'s budget, as card 9 states.
- Test economy: the new producer, engine and recipe tests each cover a distinct behavior of their own layer, and the per-package rows of `test-budget.yaml` hold.
- Docs: `doc.go` of `burlerengine`, `shedadapters` and `loomrecipe`, `docs/overview.md`, the refusal spec, `loom-recipe.yaml` and the `loom.yaml` template all land with the code.

### [LOW] Focus-directive banner names a deleted function

**Location:** contracts/stencils/burler/burler-focus-directive.md:3
**Issue:** The banner says `internal/burlerengine`'s prompt composition (`composePrompt` in `prompt.go`) is the consuming call site.
Card 10 deleted `composePrompt`; the call site is now `composeReviewPrompt`.
**Fix:** Name `composeReviewPrompt` in `prompt.go`.

### [LOW] Comment cites a deleted engine type

**Location:** internal/burlercli/cli.go:58
**Issue:** The comment says the mode, state directory and stencils directory are not threaded through `burlerengine.Result`, which card 10 deleted.
**Fix:** Name `burlerengine.ReviewResult` and `burlerengine.FixResult`, or drop the type reference.

### [LOW] Seat directive banners point at retired Burler seat stencils

**Location:** contracts/stencils/seat/seat-directive-advisor.md:4 and contracts/stencils/seat/seat-directive-chair.md:4
**Issue:** Both banners say the seat-to-seat channel wording lives here and "in the Burler seat stencils, which state a narrower channel of their own".
Card 10 retired the reviewer-fixer channel and its stencils, and the two Burler half stencils are one-seat chairs with no advisor, so no such stencil remains.
**Fix:** Drop the Burler clause from both banners.
