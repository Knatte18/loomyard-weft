# Board policy: a tapestry-sized note runs as a Motif, then raddle

The operator's rule that a board note weighed for a tapestry run goes through Motif and raddle lived only in one hub's heddle entry.
This change ships it in the `ly:board-policy` skill, so every hub's orch and board sessions follow it.

## What changes

`plugins/ly/skills/board-policy/SKILL.md`, under Tasks, gains one bullet:

- a note or bundle weighed for a tapestry run is proposed as a `motif` task, never as a tapestry task straight from the board;
- promoting the Sketch its Motif makes runs raddle, which splits it into tasks;
- raddle chooses each part's recipe, tapestry included, and a board session never changes the recipe of a task raddle filed.

Under "A session opened for board work", the split bullet now says a Sketch is split only by promoting it to its raddle task, never by hand.
Other entries are still split by hand on the operator's approval.
The old bullet let a board session split any discussed entry by hand, a Sketch included, which went against the rule that raddle splits a Sketch.

## Design decisions

- The orch role (`contracts/orchrole/orch-role.md`) does not say how a recipe is chosen.
  Its board section defers to `ly:board-policy`, so the rule goes in the skill alone and the role stays as it is.
- The existing bullets that fold related notes into one loom task, and propose darn for a small change, stay.
  Bundling still decides what one Motif covers, and darn still covers small changes.
- Raddle's stencil, including its weighing of tapestry's fixed start cost, is unchanged, as the task asks.
- The hub's own `HEDDLE.md` entry for the rule is in the hub's bolt, not in this repository, so this change leaves it alone.
  Once this lands, the operator can drop that entry.
