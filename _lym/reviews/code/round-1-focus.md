---
round: 1
focus:
  - "internal/hubgeom/buttons.go pairIsBeam: does boardengine.ReadTasks return no error and no entries when the board file or directory is absent, so a hub without a board still reads as task and does not fail reed's boot?"
  - "Goal's Expected line asks for the beam group and the beam worktree as button dir: does the diff fix the button dir (@lym_button_dir pointing at the prime worktree in the Seen section), or only the place?"
  - "Goal's Seen section suggests a batten run classifies the session as a task session: does a code path outside hubgeom's buttonContext (reedengine place or dir selection, batten session setup) still produce the task group for a beam under batten?"
  - "pairIsBeam matches the board entry by task.Slug == l.WorktreeName: is that the right key for a beam pair, and can a non-beam task sharing the slug or a differently named worktree read wrongly?"
  - "hubgeom_test.go TestReedGeometry_ButtonContext: do the new board rows cover behavior the existing rows do not, and is the board-absent path covered?"
---

Seed focus for round 1 from the Goal and the two-commit diff over `hubgeom`.
The decision record's other sections are empty, so the Goal is the measuring stick.
