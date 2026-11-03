---
round: 1
focus:
  - "internal/shedbuild Expand: does nested-block rewriting (innermost first) set every empty on_done and empty skip target to the innermost enclosing block's exit, and does it enforce each D2 refusal (unknown or nil resolver, self-inclusion, name and segment collisions, edges leaving the sub-recipe, empty on_done on an outermost block row, on_done naming the block or its inlined rows, sub-recipe setting contract, verify_source, amendment_target or profile_recipe)?"
  - "internal/shedengine entry.go, goto.go, reroute.go, progress.go: does every reader that routes or reconstructs a skipped entry take SkipTo when set and OnDone otherwise, and does a goto onto a block row bypass its gate while a re-launch onto it clears the entered row and evaluates the gate?"
  - "internal/shedengine history writes: does every entry for an inlined row (shed-appended, skipped, goto and relaunch) carry the Block path, with the field omitted when empty so older status files still read?"
  - "internal/shedbuild Parse: are all D1 refusals present (block row setting engine, config, on_stuck, on_reroute, segment, max_bounces or entry_gate; missing block or block_gate; hand-written engine Block), each naming the row and a way forward, and is an empty on_done allowed on a block row?"
  - "internal/shedrecipe: is Block a single Constructor entry returning Done, is BlockGates a closed table reached by lookup, and does Build refuse an unknown block_gate naming the row and the shipped checks?"
  - "internal/loomrecipe: do the build, routing projection, row list, RecipeEngines and the bouncer and finalize row helpers all read the expanded recipe through the one resolver table, and does the profile_recipe accessor serve every reader of a task's profile block?"
  - "The motif and arras recipes: do motif's rows equal tapestry's Discussion rows (config included, bar the Discussion-Bouncer's empty on_done), does arras expand to tapestry's rows plus the Motif block, is motif absent from shedrun.RecipeNames, and is tapestry's recipe file unchanged?"
  - "Sites that name tapestry (battencli wire.go and runsettings.go, reportengine runs.go and timingspans.go, loomrecipe recipekeys.go, shedrun seed): is arras accepted and wired at each, including a phase for the Motif row, with no site left that tests RecipeTapestry alone?"
  - "Status and progress (shedverbs human.go, shedcli table.go, shedengine status.go, progress.go): does a block row's Done show as entered, does an inlined row show its block beside its name, and does the count number the block row as its own step and jump from it to the exit on a pass-over?"
  - "Commit messages and the change summary: does any Pattern-Departure trailer name a real entry that forbids the answer key's way, and does every claim in _lym/landing/summary.md hold against the diff?"
  - "Test economy: do new tests duplicate coverage already given by a public-surface test, and is any added behavior (for example a skip-target reader or a refusal) left without a test?"
---

Seed focus for round 1, set from the decision record, the plan overview and the diff stat of `e85d20aa..HEAD`.
Entries name where to look and which question to settle; none pre-states a verdict.
