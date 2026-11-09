# The arras recipe and its split perch

`arras` is a new loom recipe for a change too large for one tapestry.
It discusses the task, splits the approved decision record into board entries, and files them.
A review perch over the split, the split perch, approves the split in place of the operator, so the run lands no code and needs no operator approval of the proposal.

## What changes

- `contracts/recipes/arras-recipe.yaml` declares `contract: loom` in its record-only form.
  Its rows, in order:
  - the `Motif` block row, with `block_gate: undiscussed`;
  - `Raddle-Write`, with its `raddle` gate;
  - the split perch: `Raddle-Bouncer`, `Raddle-Burler-Review` and `Raddle-Burler-Fix`, in segment `Raddle-Review`;
  - `Raddle-File`, whose Stuck returns to `Raddle-Write`.
- The split perch reviews `_lym/raddle/proposal.yaml`, with the decision record as read-only context:
  - its rows run on the generic Bouncer, BurlerReview and BurlerFix engines, with `role_prefix: split` and run directory `split`;
  - the fixer is cold, runs `fix-scope: overlay` and carries the `raddle` gate, so a fixed proposal passes every proposal check before the next round;
  - the Bouncer sets `escalate_decisions`, and the fixer row sets `decisions_escalated`;
  - the Bouncer sets no `carry_over`, `approve_seam`, `commit_seam` or `subject`.
- Because the Bouncer's artifact path names the proposal, raddle's existing rows test records `perch` mode for an arras run.
- `loom.yaml` gains the split perch's keys under perch name `split`:
  - `split_max_bounces` (2) and `split_hard_max_bounces` (0, taking `review_hard_max_bounces`);
  - `split_auto_accept` (false);
  - `split_reviewer_model`, `split_fixer_model` and `split_judge`;
  - `split_reviewer_fan` and `split_fixer_fan`.
  `split_auto_accept` is a guard key, as `reviewbudget` registers every perch's auto-accept key, and both fan keys are guards like the Discussion perch's.
  An empty `split_fixer_model` takes the `raddle` key's model.
  The split judge's seed falls back to `seed` and then the split judge, since the segment has no seed key of its own.
- `loomcli` maps segment `Raddle-Review` and row `Raddle-Bouncer` to perch `split`.
  That gives the segment its models, budget, hard cap and circling lookup.
- New rubric `contracts/stencils/loom/loom-rubric-raddle-review.md`, embedded and registered with the other loom rubrics, tells the reviewer to read the whole record, `Added after Discussion` entries included.
  It judges each part's scope and contract against the decisions, the seams along `depends_on`, the order and `order_departures`, each part's recipe, profile and cost lines, and one task against a family.
  It names the proposal spec's mechanical checks as the `raddle` gate's and does not re-derive them.
- `arras` is registered wherever `motif` and `raddle` are:
  - `shedrun.RecipeArras`, with the seed test that pinned `arras` as a former name reversed;
  - loomrecipe's embedded recipe table, `RecipeEngines` and `BouncerRunSubdir`;
  - shed's arming table;
  - batten's admitted child recipes and their refusal;
  - the run profiles template: an `arras` block in each ladder profile sets the Discussion and split perch rounds, and `defaults` maps `arras` to `solid`.
- The arbiter describes an arras run as a task's Discussion followed by its split, not as a split of a stored Sketch.
- `docs/overview.md`, the glossary and the package docs of `loomrecipe`, `loomcli`, `shedrun` and `runprofile` name the recipe and the perch.

## Design decisions

- The perch name is `split`, so its keys read `split_*`, and its segment is `Raddle-Review`, the segment the raddle perch-mode fixture already uses.
- The perch re-arm needs no new code.
  A Bouncer with no subject seam keeps no settle record, so its next entry over a settled verdict clears its run directory and re-seeds from round 1.
  Its earlier Done ended the segment's bounce episode, so the new proposal gets the full budget.
  A run-level test pins this: with a budget of 1, a Raddle-File return reaches a second review round and the run ends.
- The fixer's raddle-key fallback applies only while `raddle` is set, which every loaded config guarantees.
  A bare `Config` built in a test keeps `fixer_model`.
- The split perch has no assistant reviewer and no seed key; the record lists neither among the segment's keys.

## Tests

- `internal/loomrecipe/arrasrecipe_test.go` pins:
  - the rows and their routing;
  - `Motif`'s skip target;
  - the Discussion approval row;
  - the split perch's absent seams and its fixer's gate.
- The same file drives two run-level cases over the sequence fixture with fake Raddle-Write and Raddle-File rows:
  - a filing on the first proposal;
  - a Raddle-File return whose new proposal the perch reviews again.
- The existing recipe-list tests now include arras: row names, entry numbers, structural check, interrupt policies, recipe keys, split-perch detection, fan settings, the approve seam rule and the decision switches.
- The loomcli tests cover the arbiter description and the Raddle-Review models, the stencil rubric test covers the new rubric, and the shedcli, battencli, runprofile, configsync and configreg tests take the new name and keys.
- No test was run in this session; the verify gate runs them.

## To check by hand

- Run an arras task end to end on a real hub.
  The run-level tests fake Raddle-Write and Raddle-File, so nothing here ran a real raddle session or filing under the split perch.
- Read the rendered Raddle-Review rubric in a split perch round's prompt.
