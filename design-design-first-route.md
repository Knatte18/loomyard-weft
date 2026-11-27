# Design-first route: motif in place of arras, Sketch follow-up and a tuned raddle split

`design-first-route` · Task · enhancement, stencil, orch, board, loom · starting · Seed-Child

## Aim

Motif, then the Sketch's raddle run, is the one design-first route, and it works under autonomy:
no step of it waits on the operator except raddle's split approval, which stays the operator's.

## 1. Hide arras (operator, 2026-11-27)

Orchs still propose `arras`, though it is never used; the operator does not trust raddle's split enough to let a run split an entry on its own approval.
Arras is no longer an option anyone is offered. Its code stays, but nothing visible names it:
the shipped orch role, the `ly:board` and `ly:board-policy` skills, help text and recipe listings.

- Rewrite the board-policy passage that proposes an arras task for a tapestry-sized note to propose motif.
- Drop arras from the recipe lists in the board skill, the orch role and `lym board` help.
- A board write naming `recipe: arras` is refused, with a hint naming motif (operator, 2026-11-27).

## 2. Follow up every Sketch (operator, 2026-11-27)

A motif run ends by leaving its entry as a Sketch, a note that never runs, and nothing follows it up.

- When a motif run lands, the orch is told.
- Outside autonomy, the orch puts the Sketch to the operator: promote it to its raddle run now, or say what it waits for.
- Under autonomy, the orch promotes the Sketch to its raddle run itself once nothing it waits for is open,
  and gives it `depends_on` for what it waits for, so the board shows why.
  The raddle run's split approval still goes to the operator, as a merge-ready beam does, and is not a halt the orch settles.

## 3. Tune the raddle split

`docs/research/raddle/raddle-split-quality.md` read every raddle run of this hub up to 2026-11-27.
The most common revision is a missing dependency on work already running or released;
corrections to part count and recipe are about size and cost.
Apply its section "Proposed stencil changes" to `contracts/stencils/raddle/raddle-template-agent.md`
and `contracts/stencils/loom/loom-rubric-raddle-review.md`:
order and dependencies, part count and size, recipe and profile, cut lines, missing or invented parts, wording,
superseded decisions, size estimates, where the entries are filed (visibility only), and the split perch's rubric.
Letting the operator choose the filed project's level needs code and spec, and stays out of this task.
