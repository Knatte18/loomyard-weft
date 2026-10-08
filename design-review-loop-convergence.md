# Review loop: a lighter convergence rule, a three-round default budget, models per segment

`review-loop-convergence` · Task · bug, enhancement, burler, loom · running · Plan-Bouncer

## What happened

On 2026-10-08 the Discussion review segments of `plan-glyph-precision`, `shuttle-state` and `reed-view` all ran their five rounds and escalated to the parent with cause `budget`.
In each, the round-5 reviewer's verdict was APPROVED with no BLOCKING finding, every earlier key was resolved, and each round raised fresh keys on edge cases rather than repeating old ones.
The parent then recorded the remaining design calls as decisions and accepted, which another round would not have changed.

The judge's decision rule (`decisionRuleMarker` in `internal/shedadapters/bouncerprompt.go`) says `CONTINUE` while the latest round carries any `design` finding at MEDIUM or worse, and the judge stencil counts no fix as convergence, because a fix is new text no fresh reviewer has seen.
Every round's fixes so yield a fresh full review and fresh MEDIUM keys until the budget runs out; the reviewer's own verdict carries no weight.

Measured over the ten latest landed runs (2026-10-08), review loops take about 57% of a run's agent time (median about 24 min Discussion, 27 min Plan, 34 min Webster review, against 39 min of Webster execution), with median rounds 3.5, 2.5 and 2.5, and 5 of 10 Discussion reviews escalated on budget.
Nearly all of a round is the reviewer pass; the judge takes under a minute.

## Proposal

- **A lighter convergence rule:** `CONVERGED` when the latest round carries no BLOCKING finding and no MEDIUM `design` finding on a key that was already open in an earlier round.
  New MEDIUM keys in the closing round are not dropped: they go to the next producer through the decision record, as the parent did by hand on 2026-10-08.
  BLOCKING always continues.
- **A default budget of three rounds (operator, 2026-10-08):** the template default of `review_max_bounces` (`internal/loomengine/template.yaml`) drops from 5 to 3; a spent budget still escalates to the parent, who decides what to do ("it is mostly fine after three rounds").
  The prime's own `loom.yaml` already carries 3.
- **Models per segment:** `loom.yaml`'s `review` and `fix` keys take one model-spec or a per-round list, but one setting serves every review segment.
  The task lets each segment (Discussion, Plan, Webster review) set its own review and fix models, with today's keys as the default.
- **An option to start the fixer when the review is done:** the fixer starts beside the reviewer and polls `lyx burler await-review` so its orientation overlaps the review (572ab84e8).
  When a review runs longer than the prompt-cache TTL the fixer's cache expires while it waits, and each await poll is a paid turn.
  The task adds a config option to start the fixer only once the review file exists instead; today's parallel start stays the default (operator, 2026-10-08: only the option for now).

The lens fan for Discussion and Plan is task `review-lens-fan`, which waits on `shuttle-state`.

## Circling decisions resume the run (from issue #446)

Imported from [issue #446](https://github.com/Knatte18/loomyard/issues/446).

After `lyx loom circling accept plan-glyph-precision` (Discussion-Bouncer round 5, cause budget, under a watching batten run), the verb printed `"resume":"lyx loom start"`, and `lyx loom resume` in the task worktree then refused: "the run is awaiting a decision, which resume does not give; run \"lyx loom approve\" or \"lyx loom reject\"", although the decision was recorded and approve/reject are PR-Gate verbs.
The run moved on only after the driver stepped it.

Expected: after `circling accept` or `continue` under a watching batten run, both outputs name the same next step, and `loom resume` either resumes on the recorded circling decision or says the decision is recorded and the driver will step it.
The hub's role tells it to run `lyx loom resume` after a circling decision.
