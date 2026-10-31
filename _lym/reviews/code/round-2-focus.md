---
round: 2
focus:
  - "In internal/loomrecipe/loomrecipe.go, read the map TapestryRetiredRows returns: does it map the old row name Loom-Preflight to loomshed.NameTapestryPreflight, so a tapestry run whose status is parked on Loom-Preflight resumes, and does TestRenameRetiredRows in internal/loomcli/wiring_darn_test.go pin that entry?"
  - "In internal/shedrun/seed_test.go TestValidateRecipe and the unknown-recipe rows of internal/shedcli/cli_test.go, does a test assert that the old recipe value loom is refused as unknown and that the refusal lists tapestry?"
  - "Search README.md, docs/overview.md and tools/sandbox for the old recipe value loom used as the default recipe, the empty-recipe meaning, or a seed's recipe field (including the spaced form in tools/sandbox/SANDBOX-CORE-SUITE.md); does any site still carry it where the text means the recipe rather than the general run or the loom config module?"
  - "In internal/lenses/doc.go, internal/reviewbudget/doc.go and contracts/specs/shed-recipe-spec.md, does each use of loom or tapestry match what it means: tapestry for the recipe, loom for the config module and its fan key grammar?"
---

Round 1 left one BLOCKING design finding and several rename leftovers; the focus asks whether each site now reads correctly.
