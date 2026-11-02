# Report the plan-split replay: both inputs join in round 1

This change adds the replay results to the plan-split research report and changes no code.
`plan-split-tune` landed round 1's tuning of the plan-split writers without replaying it, because stencils reach a run only from the hub's Bolt copy, which takes only what lands on `main`.
The replay ran afterwards on the installed `lym`, built after that landing.

## Result

Both bench inputs, `warp-visibility` and `run-verbs-shared`, joined their parts into a plan in round 1 and passed their gates with no hand edit of `parts.md` or the part plans.
The loop therefore ended after round 1 of its cap of 2, with 2 split writings and one Plan-Review round per input.

- `warp-visibility`: one `plan-part` refusal (`handle-dangling` on the `entry-merge` part), cleared by the part writer on the gate's re-prompt; 36 cards; writing cost $18.69.
- `run-verbs-shared`: no refusal; 37 cards; writing cost $29.36.
- Plan-Review found no blocking finding on either plan (8 and 12 findings).

The three defects the bench's hand edits repaired did not recur.
One run per input is a direction, not an effect, and the report says so.

## What changed

- `docs/research/plan-split-tune-2026-11-02.md` now holds the replay's method, the round-1 table, the Plan-Review table, the card overlap and a reading, and its title and opening paragraph cover both tasks.
- `docs/research/plan-split-replay-2026-11-02/` holds each split plan's `plandag` report and its Plan-Review round.

## Design decisions

- The 2026-11-02 author rewrite changed every commit SHA, so the bench's SHAs no longer resolve.
  The orch mapped each to its rewritten equivalent by date, and the report records both sets.
- The runs were set up as the bench set them up: a driverless seed, a hand edit of each run's `status.json` onto `Parts-Write`, and pauses after `Parts-Plan-Write` and `Plan-Burler-Review`.
  The orch authorized the edits inside the two scratch worktrees only.
- The Plan-Review fixer was not run, since the cap names one review round per input; the report lists the findings as unresolved.
- No tuning was made, since round 1 met the exit condition, so no change waits to land and the report names no next check.
