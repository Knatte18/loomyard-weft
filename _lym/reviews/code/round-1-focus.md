---
round: 1
focus:
  - 'internal/shedbuild/expand.go: does the run-ending rule, applied to the resolved block exit, match D3 for nested block rows, and do all inlined edges (on_done, skip target, other routing) of a run-ending block resolve correctly? Which of these cases do the tests cover?'
  - 'internal/shedbuild/loomcontract.go: does every later clause, and every reader of the record row outside this file, treat a seed-only recipe as the contract intends, including a seed row off the on_done path from the entry?'
  - 'loomrecipe.RecordEngine now returns RecordSeed for jacquard: trace each caller and settle what it does for a jacquard run. Separate the work the amended entry takes in this change from the work it leaves to later parts.'
  - 'Jacquard in the recipe vocabulary, and Preflight and Record-Seed now following Discussion in tapestry: does any code, config lookup, profile, report or batten path keyed on the tapestry name, on Preflight being the entry, or on row order behave wrongly for a jacquard run, a new tapestry run or an in-flight run resumed by row name?'
  - 'internal/loomshed/recordseed.go and recordwrite.go: check each write, keep, commit and Stuck path against D4, including a missing support log, a failing seed read and the cancel check.'
  - 'Record-Seed Stuck reason and recordSeedDeps in internal/loomcli/wiring.go: run each named command against the actual verbs and flags. Do the sequences work in the stated order and directory, with the filled slug, seed path, driver and parent (empty or shell-special values included)? Is the branch between the Discussion goto and the re-run chosen correctly per recipe?'
  - 'contracts/recipes: compare jacquard-recipe.yaml and the new tapestry-recipe.yaml with the base tapestry file. Were rows, routing, segments, gates, comments and top-level keys moved unchanged, and are the tapestry and motif headers and other comments still true after the move?'
  - 'Test changes across the range: check each edited existing test for widened or removed assertions against the Loosened-Check rule, including whether the trailer names a measured cause. Do the new walk, contract and Record-Seed tests check what their names say and what the entry delivers, without redundant coverage?'
  - '_lym/landing/summary.md: check each claim against the diff, read the whole text against the Names of the tooling bullet, and check the Merge danger door and blast radius against what the diff can break (recipe vocabulary in stored seeds, changed graph for in-flight runs, batten).'
  - 'Repository rules: do the constraints document and the docs it names for a new recipe, engine, refusal or row name require changes the diff lacks, and are changed doc comments and file headers accurate?'
---

Seed focus for round 1.
The decision record stands alone (no plan directory), so the measuring stick is the board entry under `## Goal` as amended by its `Added after Discussion` entries.
The `D1` to `D4` sections of the record are the writer's claim, to be checked against that entry.
