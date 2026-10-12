# Fixer report — discussion round 4

## Changes

- F1 (fixed as proposed): D4 adds `discussion_producer` to loom's `LocalDenied` list, and its reason line now says that `twill` makes the Discussion-Write writer fork one reviewer per lens of burler's `twill-discussion` fan, so a local `single` or `seats` switches off a hub-chosen review, untracked.
  The acceptance row on refused local keys names `loom.discussion_producer`.
  The premise holds against `internal/loomengine/twill.go` (`twillFan`) and the `discussion_producer` comment in `internal/loomengine/template.yaml`.
  `support-log.md` Q5 is unchanged: its wording already covers reviews and lens sets, and it already names round 4.
- F2 (fixed as proposed): D1 states that the layers are merged unexpanded and the merged document is expanded once, through `envsource.Build` of the reading worktree's anchor for a per-worktree module and of the board dir for a hub-wide module, and that no layer is expanded against its own directory's `.env`.
  A new acceptance row says a `${env:...}` marker the worktree file inherits from the repo default expands against the reading worktree's `.env`.
- F3 (fixed as proposed): the strip acceptance row adds the reverse case: a worktree marker over a repo-default literal equal to the walk's expansion of that marker, which is kept.

## Disputed

None.

## Deferred

None.
