# Per-task run settings on the board entry

`task-run-settings` · Note · feature, board, batten, loom

## Seen (2026-10-11)

The orch sets `review_max_bounces: 2` by hand in a small task's pair `loom.yaml` right after batten creates the pair, and must first check that the pair exists, since a run can stall at Worktree-Create.
Landing then warns that the task branch changed a per-worktree config file, which needs nothing but costs a look each time.
The board entry already carries one run setting, `recipe`.

## Idea (operator, 2026-10-11)

Run settings such as the number of review rounds per perch (Discussion, Plan and Webster review segments) and for darn's review are part of the task: the board entry carries them, and batten applies them to the pair when it creates it.
Starting a ready task then needs no hand step, which `orch-split` needs for a mechanical run driver.

## Open

- Which settings a task may carry: a closed list of keys, or any key of the recipe's config module.
- Where batten applies them, so landing does not flag them as config changes meant for the parent.
