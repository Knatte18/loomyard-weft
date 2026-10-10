---
round: 3
focus:
  - "Card order across the Env seams: for every func-typed field a card adds to Env (for example Env.DiscussionTable, a shedadapters.TableSource), check whether internal/loomcli's TestWire_EverySeamFilled (internal/loomcli/wiring_seams_test.go, using envkit.NilSeams) stays green after that card alone, either because loomcli's wire fills the field in the same card or because the card adds and a later card removes an intentionallyNil bridge entry."
  - "Targets versus pinned tests: for every card that changes what a function returns (for example loomrecipe's RecipeEngines gaining the discussion seats engine), check that each existing test pinning that function's result (for example TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet in internal/loomrecipe/recipe_test.go, which compares against the engine column of wantProducerTable) is in the same card's Edit list, so the plan's go test ./... stays green after each card."
  - "Fidelity to the decision record's acceptance criteria: check that the criterion that the discussion gate and the parent-review brief read only the two discussion files, with an advisor notes file beside them, is pinned by some card's planned test row."
---

Focus entries restate the sites they depend on.
No entry caps severity or pre-states a verdict.
