```yaml
lens: fix-consistency
outcome: reported
```

The fix-consistency lens checked the change set's agreement with sibling cards, decisions and cross-references. It found one issue.

- LOW — `_lym/plan/00-overview.md:72`, Shared Decision `pattern-headroom`. The overview still says card 4 "moves each dropped rule verbatim into its background file". The card 4 fix for F18 now says no rule text moves and no background file changes. It also dropped the three `pattern/*.md` files from card 4's Extends. The overview twin was left as it was, so it now contradicts card 4.
  Evidence: overview line 72 against card 4's new text, "Every rule the shortened lines drop is already stated in its entry's background file, so no rule text moves and no background file changes".

The other changes I checked agree with their surroundings:
- Card 13 now declares `shedrun.RecipeRaddle`. Card 23 cites it as card 13's constant and lists it under Uses. Card 24's `plan:` use comes later than card 13.
- The project type label agrees across cards 10, 12, 16 and 22.
- The `filing-level` Shared Decision and card 16 both cite the Added after Discussion entry.
- `internal/loomcli/doc.go` is now Used only by card 25, which comes after card 23 where it is a target.
- Card 25's `Notify` re-sign covers every fake and caller in the code: `world_test.go:180`, `pawlwiring.go:137`, `awaitRun.notify`, `validPawlDeps` and `envkit.FullEnv`.
- Card 14's reworded bound still matches card 24's reference to it.

Tool calls:
1. Grepped the plan cards for twins of the changed text (`loomcli/doc.go`, `envkit#FullEnv`, `SketchRecipe`, "moves each dropped", the pattern background files). My context held only the cards I had read, not every card in the plan.
2. Grepped `internal` for pawl `Notify` fakes and callers, and the overview for card 4 and background-file references. My context lacked the full set of `Notify` construction sites.
