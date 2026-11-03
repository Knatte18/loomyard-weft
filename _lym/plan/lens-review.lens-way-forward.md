```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan in `_lym/plan`. I checked each refusal, stop or recovery step the cards add against the next step it names.

**MEDIUM — Card 8 (`profile-recipe`), D8 / the arras seeding path.** For a task whose recipe names no embedded recipe, card 8's new refusal tells the operator to fix the wrong thing, and it fires before the refusal that names the right fix.
- Card 8 resolves `settings.Recipe` through `loomrecipe.ProfileRecipe` inside `prepareRunSettings`. An unknown name is wrapped in `ErrInvalidRunSettings`.
- Seed-Child calls `writeRunSettings` before `WriteSeed` (`internal/battenshed/seamchild.go:74` vs `:93`).
- So a task whose recipe field names no embedded recipe (for example a typo) now stops at the run-settings refusal. Its way forward is "correct the task's profile or run_settings … or the profile itself with `lym config profiles` … then `lym batten run`" (`seamchild.go:143-144`).
- That step cannot succeed from that state: the profile is fine and the recipe field is what is wrong.
- Today the same task gets `unsupportedChildRecipeRefusal`, which names the admitted recipes (`internal/battencli/wire.go:537-538`), via `childRecipeRefusal` at `seamchild.go:94`.
- Card 8 also plans a `TestPrepareRunSettings` row that pins the misleading path.
- Fix: when `ProfileRecipe` refuses, keep `settings.Recipe` unchanged, so `Default` returns "" and the seed's own refusal names the way forward. Otherwise the refusal must name the task's recipe field and the admitted recipes.

**LOW — Card 5 (`shedbuild-block-build`), refusal for an unexpanded block row.** The new refusal is unreachable for a loom recipe, because an earlier check fires first with an unrelated way forward.
- `Build` runs `checkLoomContract` before any per-row work (`internal/shedbuild/build.go:38-42`).
- An unexpanded `arras` has no `DiscussionWrite` row, so it is refused as "0 record rows; way forward: give the recipe exactly one record row". That way forward is wrong: the recipe is fine and needs expanding.
- Fix: put the unexpanded-block check ahead of the loom contract check.

**LOW — Card 4 (`shedbuild-expand`), refusal texts.** The eight `Expand` refusals say they name "a way forward" but the card never states any of those texts. The planned rows "assert … the way forward" without a value to assert.
- PATTERN-refusal-way-forward needs each refusal to name a way forward that a test reaches.
- Each text should be stated, for example "rename the inlined row" or "set the block row's on_done".

**LOW — Card 1 (`shedengine-block-routing`), `validate` refusal.** The new message `has SkipTo %q naming no producer in the list` names no way forward. It matches the existing `validate` messages, which also name none.
- It is reachable only from a hand-built list, so it reads as a fail-loud wiring guard.
- The card should say it is one in `refusal-spec.md`'s sense, or add a way forward.

Tool call made:
- Read `internal/battenshed/seamchild.go:55-150` and grepped for `ErrInvalidRunSettings`. My context lacked the order of Seed-Child's run-settings write versus its seed write, and the way forward that run-settings refusals carry.
