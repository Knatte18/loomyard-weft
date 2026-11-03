---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/plan/08-profile-recipe.md:18
    summary: Card 8 Intent names unsupportedChildRecipeRefusal as the refusal an unembedded recipe name reaches, but ValidateRecipe refuses it first with ErrUnknownRecipe
    fix: In card 8's Intent, rewrite the sentence that begins "When ProfileRecipe refuses" so it names the refusal an input actually meets. Say that a task whose recipe names no embedded recipe, with no profile named, passes prepareRunSettings with its settings alone and is then refused by the WriteSeed seam through shedrun.ValidateRecipe, which wraps battenshed.ErrUnknownRecipe and names the available recipes; and that a task naming a profile is still refused by profiles.Settings as it is today, because the profile holds no block for that name. Drop the clause saying the run reaches unsupportedChildRecipeRefusal, which only a known, non-embedded-or-unsupported name such as batten reaches, and for which ProfileRecipe does not refuse. No target or verify line changes, so no validate-plan overlay is needed.
---

### [LOW] Card 8 names the wrong refusal downstream of the kept recipe name

**Location:** `_lym/plan/08-profile-recipe.md:18`

**Issue:** The focus question is whether any input reaches a `ProfileRecipe` refusal inside `prepareRunSettings`. One does, so the branch and its third test row are not a defect.

The focus text puts `WriteSeed`'s recipe refusal ahead of `PrepareRunSettings`. The code runs them the other way round. `seedChildProducer.Call` reads the board type, then `writeRunSettings` calls `deps.PrepareRunSettings` (`internal/battenshed/seamchild.go:74`, `:141`), then reads the driver, and only then calls `deps.WriteSeed` (`:93`). `WriteSeed` in `internal/battencli/wire.go:1391` is where `shedrun.ValidateRecipe` and `childRecipeSupported` run. The board stores any recipe string: `boardengine/task.go:316` and `:325` only constrain the project recipe. `ReadBoardType` returns that string unchecked, so a task with `recipe: bogus` reaches `prepareRunSettings` before anything validates it.

With no profile named and no default for `bogus`, `ProfileRecipe("bogus")` refuses. The keep-the-name branch is therefore needed: without it the refusal would turn a task that today reaches the seed write into a run-settings failure. The card's third test row (recipe naming no embedded recipe, no profile, settings written alone, no refusal) is the reaching input. The branch and the row are both justified.

The Intent describes the downstream result wrongly, though. It says such a task "reaches Seed-Child's seed write, whose `unsupportedChildRecipeRefusal` names the admitted recipes, as it does today". `WriteSeed` calls `shedrun.ValidateRecipe` first (`wire.go:1392`), so an unembedded name is refused with `battenshed.ErrUnknownRecipe` and "available recipes", which `childRecipeRefusal` turns into the "unknown recipe name" stuck reason (`seamchild.go:174`).

`unsupportedChildRecipeRefusal` is reached only by a name that is in `RecipeNames` but fails `childRecipeSupported`, today `batten` (`seed.go:85`, `wire.go:532`). For `batten`, `ProfileRecipe` does not refuse, because it is an embedded recipe.

The phrase "rather than a run-settings refusal pointing at the profile" holds only when the task names no profile. With a named profile and an unembedded recipe, `profiles.Settings` still refuses with "holds no bogus block", before and after the card. A fixer or implementer reading the Intent literally could add a test, or a branch, for a path that the code never takes.

No other card-8 input is left without a reaching test. `ProfileRecipe("arras")`, `("tapestry")`, `("")`, `("darn")` and the unknown name are each a row in `TestRecipeKeys`, and `prepareRunSettings` never receives an empty recipe (`battenshed/deps.go:319`).

## Rejected

- Card 8's refusal branch is unreachable: rejected, as above; the focus text had the call order reversed.
- Any further finding against cards 1-7 and 9: I skimmed the cards against the decision record and the rubric (card order, `Uses` edges, test economy, no `Verify:` lines, the declared PATTERN text in card 2) and found no defect worth recording this round. I did not read the prior round's files.
