# Dead strands from before a reboot hold their names and fail a resumed row

`reclaim-dead-strand-names` · Note · bug, reed, shuttle

Imported from [issue #799](https://github.com/Knatte18/loomyard/issues/799).

## Seen (orch, 2026-10-21, OSL-1046)

After a machine restart, both in-flight runs were resumed from the prime with `lyx batten run <slug> --window`.
Reed still listed the strands from before the restart as dead (`live: false`), holding their names.

- `test-tiers`: Plan-Burler-Fix re-spawned its fixer as `ly:test-tiers:plan-burler-fix-2`, and the row failed with `seatengine: strand "ly:test-tiers:plan-burler-fix" started as "ly:test-tiers:plan-burler-fix-2", because another strand holds that name; way forward: run "lyx reed remove --name ly:test-tiers:plan-burler-fix", then re-step the row`.
- `tuned-defaults-template`: Merge-In's conflict agent came up as `conflict-2` beside the dead `conflict` and carried on.

## Workaround

`lyx reed remove --name <dead strand>` in the task worktree, then `lyx loom resume`.

## Expected

A strand whose pane is gone, such as every strand after a reboot, does not hold its name: a resume or a re-spawn reclaims the name of a dead strand without a manual `lyx reed remove`, and the row does not fail.