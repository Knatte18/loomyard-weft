# Ship the rule: a tapestry-sized note runs as a Motif first

The `ly:board-policy` skill now tells a board session to propose a note or bundle weighed for a tapestry run as a `motif` task.
It no longer sends it to tapestry straight from the board.
The Motif stores the approved decision record and makes the entry a Sketch.
Promoting the Sketch runs raddle, which splits it into tasks.

The rule used to live in one hub's heddle entry.
It belongs to lym, so every hub gets it from the shipped skill.
The reason is cost: the tapestry Plan phase costs about $35 a run, half of it review and fix rounds.
Motif then raddle splits a large change before any plan, as the fabricengine rewrite did into 11 darns.
The measurement is in `docs/research/loom/writer-cost-remeasure-2026-11-08/report.md`.

## Design decisions

- Raddle's shape choice is unchanged.
  It still chooses each part's recipe, tapestry included.
  The skill says that choice stands: a session never changes the recipe of a task raddle filed on its own judgment.
  The operator may still name darn for such a task.
- The skill's rule on splitting entries now says a Sketch is split by promoting it to its raddle task, never by hand.
  Other entries are still split on the operator's approval.
- The change is confined to `plugins/ly/skills/board-policy/SKILL.md`.
  It adds one rule to the recipe-choice list and amends one line of the splitting rule.

## To check by hand

- Nothing else in the shipped orch role or stencil text names how a recipe is chosen.
  Confirm no shipped sentence still sends a large change straight to tapestry.
- No automated check covers the skill text, and no regression check ran.
