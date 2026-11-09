# Add the arras recipe and its split perch

Adds the `arras` recipe: a task runs Discussion, then splits the approved decision record into board entries, and the split is reviewed before it is filed.
Until now a Sketch's split ran as the `raddle` recipe with no review of the proposal beyond the mechanical `raddle` gate.
`arras` joins the two flows so one run goes from a task's Discussion to filed parts, and the run lands no code.

Closes nothing: the board entry names no issue.

## What changes

- `contracts/recipes/arras-recipe.yaml` (new) declares `contract: loom`.
  Its rows are the `Motif` block (`block_gate: undiscussed`), `Raddle-Write`, the split perch, and `Raddle-File`, whose `on_stuck` returns to `Raddle-Write`.
  It has no `Record-Write` row; the block's `DiscussionWrite` is the one record row.
  An arras run restarted after its Discussion approval skips Discussion and resumes at `Raddle-Write`.
- `arras` is registered wherever `motif` and `raddle` are: `shedrun`, `shedcli`'s recipe table, `battencli`, `loomrecipe`, `loomcli`, `runprofile` and `configsync`.
  The test case that pinned `arras` as an unknown former recipe name is reversed.
- The split perch is three rows: `Raddle-Bouncer`, `Raddle-Burler-Review` and `Raddle-Burler-Fix`, on the generic Bouncer, BurlerReview and BurlerFix engines.
  It reviews `_lym/raddle/proposal.yaml` with the decision record as read-only context.
  The fixer is a cold `BurlerFix` session with `fix-scope: overlay` and the `raddle` gate, so a fixed proposal still passes every proposal check before the next round.
- The new rubric stencil `contracts/stencils/loom/loom-rubric-raddle-review.md` judges the split against the whole record, `### Added after Discussion` entries included:
  each decision lands in a part that can build it, the parts' seams match, the order follows the record or lists its departures, recipe and profile fit each part, and one task or a family fits the scope.
  It names the mechanical checks of the proposal spec as already enforced and does not re-derive them.
- New `loom.yaml` keys for the segment: `split_max_bounces`, `split_hard_max_bounces`, `split_auto_accept`, `split_reviewer_model`, `split_fixer_model`, `split_reviewer_fan`, `split_fixer_fan` and `split_judge`.
  `split_auto_accept` defaults to false and is a guard-category key, as `reviewbudget` registers every perch's auto-accept key.
- The arbiter prompt describes a recipe with both `RaddleFile` and `DiscussionWrite` rows as a task's Discussion followed by its reviewed split.
- `docs/overview.md`, the weaving glossary and the `loomengine` and `runprofile` templates and doc comments cover the recipe and the keys.

## Design decisions

- The Bouncer sets no `carry_over`, so no round writes into the decision record.
  It sets no `approve_seam`, so the Discussion approval stays the recipe's one approval row.
  It sets no `commit_seam`: `Raddle-File`'s commit of the raddle directory commits the approved proposal.
- `escalate_decisions` is on, so decision-class findings go to the run's parent through the Bouncer, where the arbiter settles them first.
- `split_fixer_model` defaults to the `raddle` key's model, so the split is fixed by a model of the raddle agent's strength.
  The perch has no seed key of its own; its seed is its judge.
- A fresh entry into `Raddle-Write`, whether a return from `Raddle-File` or a goto, re-arms the perch when its last verdict settled.
  The Bouncer has no Subject seam, so it keeps no settle record, and its next entry over a settled verdict clears its run directory.
  The new proposal is reviewed in a fresh cycle with the full budget; `Raddle-File`'s halt on its third failure bounds the cycles.
- A goto into `Raddle-Write` from a perch that has not settled, such as one awaiting its parent on an escalation, opens no fresh cycle.
  The Bouncer raises its escalation again.
  Below the hard cap, `lym loom circling continue` runs the next round over the new proposal.
  At or past the hard cap the round can only be accepted, so `lym loom circling accept` files the new proposal with no round having reviewed it.
- `arras` declares `contract: loom` and verbs that serve some loom recipes refuse on others by the recipe's rows, not its name.

## To check by hand

- A decision added after the perch's last approving round and before `Raddle-File` reaches every part's record but is not reviewed against the split's seams.
  This is accepted in the design; confirm it is acceptable.
- No regression check ran, and no full arras run was driven end to end by hand.
  Run an arras task through Discussion, the split perch and `Raddle-File`, including one `Raddle-File` return, to see the perch re-arm.
