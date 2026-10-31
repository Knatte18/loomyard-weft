---
round: 4
focus:
  - "For every card that stops filling a Go field, drops a config key or moves a key between config files, search the repository's tests for readers of that field read back after a loader such as `LoadConfig` (for example `Config.PlanLenses` and the other plan fields, which card 6 keeps `yaml:\"-\"`), and check that each such test's enclosing function is an Edit target of the card that breaks it. Question: does any test outside every card's targets still fail under its card's tier0 card-scope gate?"
  - "For every card that adds, removes or reorders a recipe row (card 9 deletes Tapestry-Preflight, card 10 moves Drift-Verify ahead of Code-Bouncer, card 11 inserts Record-Write after darn's Preflight), search the tests that pin numbers `loomrecipe.EntryNumbers` derives from the shipped rows, such as `internal/loomrecipe/recipe_test.go`, `internal/battencli` overview and dobby tests expecting a numbered cell like `6.6 Plan-Write`, and `internal/loomcli/checklist_test.go` rows like `[ ] 2. Drift-Baseline`. Question: is each such test listed as an Edit target of the card that shifts its numbers, with the new numbers named in the Intent?"
  - "In card 6's Tests paragraph, check whether a named test row reaches the `CheckResolved` refusal where the plan perch's hard cap is unset (`plan_hard_max_bounces` 0) and resolves to the run-wide `review_hard_max_bounces`, while the soft key is the plan perch's own `plan_max_bounces`. Question: does the named row assert the loom prefix with the `lym config loom` route for the hard cap beside the `lym config tapestry` route for the soft cap?"
  - "In card 2, check the `TestArmFromSeed_RefusalEnvelopeKinds` row for an unknown recipe on status against the harness: a row with no `wantKind` returns early before the suffix assertion. Question: does the card's Intent state the harness change that lets the status row assert the re-seed suffix?"
---

The first two entries are the standing method question: whether each card's affected set covers indirect readers of a changed value, not only literal key text.
Concrete instances are named only as search starting points; the reviewer re-derives the hits from the repository.
