```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix set agrees with its sibling cards, the overview and the code it cites (fix-consistency lens).

no findings

- **F1 (cards 19 and 22, allowlist):** Each card adds a different import, `raddleparser` in card 19 and `boardengine` in card 22. That matches what each card's production code first imports. No other card adds either package.
- **F2 (card 25):** The new text matches the code. `TestDobbyGeometry` has a loop over a map of path names (`internal/hubgeom/dobbygeom_test.go:112`) that the new marker can join. `TestWire_BeamRecipe` compares against `hubgeom.PawlPaths` (`internal/loomcli/wiring_beam_test.go:147`). The `PawlPaths` doc comment no longer has to say "All four", and the card now says so.
- **F3 and F8 (card 21):** These changes touch only card 21. The `RaddleTable` test sentence now covers the new `Optional` rows.
- **F4 (card 12):** No other card mentions strict decoding of the proposal. Card 4 says unknown keys are tolerated, but only for Go-written usage records, which is a separate concern.
- **F5 (card 23):** `childRecipeSupported` and `unsupportedChildRecipeRefusal` already compare against `shedrun.Recipe*` constants (`internal/battencli/wire.go:555-562`), so asking for `shedrun.RecipeRaddle` there fits the code.
- **F6 (card 22):** The `SplicePart` halt is gone from both the Intent and the `TestRaddleFile` rows. Card 11's `SplicePart` still returns an error naming an unknown id, bullet or PATTERN id, which agrees with the new "returned as the step's error". The overview mentions no `SplicePart` halt. The halt after the third return, which names the goto, is a separate rule and stays.
- **F7 (card 22):** The new labels sentence matches card 16's `RaddleEntry`, whose labels are its type label plus the Sketch's module labels with no `sketch` label.

Out of scope, noted only: card 25's Edit list does not cover the shedrecipe tests that list the Pawl path fields (`internal/shedrecipe/entries_simple_test.go:100,217,225`, `fixture_test.go:70`). Once `beamUpEntry` requires the new path, those tests need it too. Round 1 left that gap and this round's change did not create it.

Tool calls:
- grep of the plan cards outside the changed ones for twins of the changed terms (`SplicePart`, the marker, `return_note`, the allowlist, unknown keys, `PawlPaths`). My context held only the six changed cards.
- grep of card 16's labels, the overview, and the discussion for the label rule and a `SplicePart` halt. None of these were in my context.
- grep of envkit and the shedrecipe tests for the Pawl path fields, to check the twins of card 25's path sentence. My context lacked those files.
- grep of battencli's recipe-name functions, to check that card 23's constant sentence fits the code. My context lacked `wire.go`.
