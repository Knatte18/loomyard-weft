# Per-task beam settings use one lookup from the worktree up

`pull_request` and `land_notice` each wired a beam value and a task value their own way:
the receiving branch's beam.yaml plus the board field `pull_request`,
and beam.yaml's `land_notice` mode plus the board field `notify_on_land`.
Both now resolve through one lookup, which `autonomy` will build on as well.

## The lookup

`dobbyengine.LookupPullRequest` and `dobbyengine.LookupLandNotice` are the one reader of a per-task beam setting, read at call time:

1. The value the task's own worktree sets applies.
   A task sets it through `run_settings`, `{"beam": {"pull_request": ..., "land_notice": ...}}`, which batten copies into the worktree's local layer.
2. A worktree that does not set it takes its parent worktree's value, up through the beams to the prime.
3. With none set, the hub's default layer applies, else the template.

"Sets" means written in that worktree's own files, its worktree file or its local file.
The new `configengine.TopLevelValue` reads them for one place's dirs alone, so a merged hub default never counts as a worktree's own setting.
`hubgeom.TaskSettingChain` tells the lookup its chain: the task's anchor, each parent's through `ParentOf`, then the prime's, with the Bolt checkout as the board.
A worktree absent on this machine is left out, so its parent's value applies.

## pull_request

- Landing reads one value, the lookup from the landing run's own worktree, a beam's landing from the beam's worktree.
  `landingshed.PullRequestDeps` has one `Mode` seam in place of `ParentMode` and `EntryField`.
- The values are `on`, `off`, `merge` and `merge-in`; `required` is retired.
  `on` wants a pull request when the origin allows one, `merge` always wants one and squash-merges it at once,
  `off` wants none unless GitHub's rules for the parent require one, and `merge-in` lands by a true merge.
  On a beam, `merge-in` reads as off.
- A missing origin fact halts only when `merge`, GitHub's rules or a beam's recorded pull request demand one.
- `lym beam merge-parent` sets its merge task's `beam.pull_request` to `merge-in` in run_settings.
- `pull_request` is reclassified from cross-pair to `CategoryNone`: it decides how a task lands, not what lands, and run_settings write the local layer.

## land_notice

- The values are `on` and `off`; `all` and `per-task` are retired, and the template default is `off`.
- Dobby reads each run's value through a new `Deps.LandNotice` seam while the run is ahead of its Land row.
  It keeps the last reading when a read fails.
- While a beam's pull request is recorded, each member still gets its landed line with the `lym beam describe` follow-up, whatever its `land_notice` reads.
- The board-side flagging, clearing and done-entry paths are gone,
  and so are `Board.SetNotifyOnLand`, `Board.ClearNotifyOnLand` and prune's rule keeping an entry for its notice.
- The dobby view marks a run with `land_notice` and no longer shows the beam.yaml mode.

## Carrying old state over

- **Board:** the fields `pull_request` and `notify_on_land` are removed.
  `Store.Load` folds a legacy key into the entry's `beam` run settings, persisted at the next save.
  A value run_settings already sets wins, and a project's keys are only dropped.
  An upsert naming either key is refused with a hint naming its run_settings form.
- **Hub config:** the reinterpretation `beam-per-task-settings` rewrites the hub's beam.yaml.
  `required` becomes `on`, `all` becomes `on` and `per-task` becomes `off`.
  It flags a retired value the prime's own beam.yaml holds, with the `lym config beam --set` that keeps its meaning.

## Design decisions

- A beam member's landed line goes out under a recorded beam pull request even when its lookup reads off.
  The old `off` mode suppressed it.
  The line carries the beam's describe follow-up, which the orch needs to keep the pull request's description current.
- A done entry no longer owes a landing notice by itself: with the board field gone, nothing marks it, and dobby reads only runs it saw ahead of Land.
- `merge-in` is accepted at every level of the chain, though only a task's own run_settings set it in practice.

## Open risks

- A task worktree's tracked beam.yaml is seeded as a copy of its parent's at fork.
  A later change to the parent's file therefore reaches the task only when the copy does not already set the key.
- A beam or prime worktree file still holding `required`, `all` or `per-task` stops beam.yaml loading there until it is fixed.
  The reinterpretation flags the prime's file, but not a beam's.
- A run started before this change has no `beam` keys in its local layer, so a board override it carried no longer applies to it.

## Merge danger

**Door:** one-way
**Blast radius:** landing
