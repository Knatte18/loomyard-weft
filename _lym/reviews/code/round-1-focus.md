---
round: 1
focus:
  - "internal/loomcli/discussionrecord.go, internal/loomcli/wiring.go, internal/loomshed/recordwrite.go: does the discussed-entry gate seed the stored record into _lym/discussion and pass over the Motif block to Drift-Baseline, and does an entry not flagged discussed still run Discussion unchanged? Check failure paths: board read errors, a discussed flag with an empty or missing stored record, a record already present on disk."
  - "internal/shedrecipe/entries_block.go, internal/shedbuild/expand.go: the new undiscussed gate in BlockGates and the change that makes Expand ignore a block recipe's top-level keys. Does ignoring them leave tapestry's config, profile_recipe and keys exactly as before, and does it hide a real validation error?"
  - "Passed-approval test reading a row inside a block (internal/loomrecipe/discussionapproval.go and its callers): does it still find Discussion-Bouncer's approval inside the Motif block for a seeded record, as the goal requires?"
  - "motif as a runnable recipe (contracts/recipes/motif-recipe.yaml, recipes.go, shedrun.RecipeNames, loom contract record-only form in internal/shedbuild/loomcontract.go): does a motif run do Discussion-Write and its review only, store the record on the board, flag the entry discussed, and land no code? Check the draft-status write for a recipe with no Finalize row against run-lock and dobby restart behavior."
  - "Row name Motif versus Cartoon: any leftover use of Cartoon in recipes, tests, docs, stencils or code."
  - "arras removal completeness: grep the whole tree for arras, including Seed-Child, Run-Shed, reportengine runs and timing spans, tokencount, recipe keys, profile_recipe, contracts/specs, docs and plugins skill text. Is any reference left, or any removal beyond what the goal licenses?"
  - "Tapestry equality pin: internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml and its test. Does the snapshot truly equal tapestry's rows before the change, config included, and does the test compare expanded rows and top-level keys rather than something weaker?"
  - "Scope: the diff touches many doc-comment-only files (arbiterstop, gateslot, landingshed, battenshed, driftcli, and others). Is each such edit a rename consequence of arras or Cartoon, or an unlicensed change?"
  - "Test economy: new tests in loomcli, loomrecipe, shedbuild, shedrecipe and reportengine against existing coverage; name the covering test for any duplicate, and name any added behavior left uncovered."
  - "Commit messages and landing summary: any Pattern-Departure trailer, and any claim in _lym/landing/summary.md that the diff does not bear out."
---

Round 1 focus, set from the decision record's Goal and the diff stat of the review range.
