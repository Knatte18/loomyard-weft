```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the change set agree with its siblings, and did any change leave a twin unchanged?

- [LOW] Card 10, the new sentence starting "Every text that says PR-Review's Stuck is always a Halt now says…". The sentence lists the header comments of `darn-recipe.yaml` and `sampler-recipe.yaml`, plus `internal/loomrecipe/doc.go`. It leaves out the tapestry twin. In `contracts/recipes/tapestry-recipe.yaml:29-31`, PR-Review sits in the "These PRODUCER rows also carry no on_stuck … and their Stuck halts the whole run just as a gate's does" list. After cards 9 and 10, a send-back Stuck reroutes there instead of halting. The card's older sentence about header comments saying "no edge out" may have meant this line, but the new list names the darn and sampler headers and not tapestry's. That makes it read as if tapestry's header needs no change. The new sentence should add tapestry's lines 29-31.
- [LOW] Card 7, the new review-row passage ("`BurlerHalfDeps` gains `Rubric`…, which `NewBurlerReviewProducer` stores"). The card lists `NewBurlerReviewProducer` and `BurlerReviewProducer.roundProfile` under Edit, but its Intent never rewords their doc comments. Those comments have gone stale:
  - `internal/shedadapters/burlerreview.go:165` says "profile is a template; each round overwrites its per-round fields, such as ReviewPath and FocusDirective". `Rubric` now joins those per-round fields.
  - `burlerreview.go:233` lists what `roundProfile` fills, and `Rubric` isn't in the list.
  - In `internal/shedrecipe/entries_burler.go:306-309`, `burlerRoundProfile`'s doc says the stencil's filled text "sets Profile.Rubric". Now the round's seam answer replaces it.

  The Intent should say these doc comments name the per-round rubric.

No conflict elsewhere:
- Card 8's help text and refusal wording agree with card 12's brief.
- Card 9's dirty-tree halt is a separate trigger, so it needs no `.scratch/` twin.
- `00-overview.md` mentions none of the changed terms.

Tool calls:
1. Grepped the overview and the sibling cards for the changed terms. My context didn't hold the overview or cards 1, 2, 4–6 and 11.
2. Grepped the recipes for halt wording in their header comments, plus `landingshed/doc.go`. My context showed only the lines that mention PR-Review, not the shared no-on_stuck header lists.
