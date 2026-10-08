# Finalize warns of per-worktree config changes that came from main

`finalize-config-warn-from-main` · Note · bug, loom, config

Imported from [issue #450](https://github.com/Knatte18/loomyard/issues/450).

At Finalize, loom warns: `task branch "<slug>" changed per-worktree config files since it forked from "main": _lyx/config/batcher.yaml (base <sha>, tip <sha>); landing does not carry them to the parent branch, so re-apply each change meant for the parent on the parent's copy`.

Seen for `shuttle-state` and `plan-glyph-precision` on 2026-10-08.
In both, the pair's `batcher.yaml` was byte-identical to the prime's: the change was main's own commit (`batcher: cautious is the template default`, c0bf19c54) arriving in the pair through the weft sync, not an edit the task made.

Expected: the check compares the tip against the parent's current copy (or excludes changes the weft sync brought in), and warns only when the task's copy differs from the parent's.
As it is, the warning fires on every run that forked before a config change on main, and the hub must diff each time to rule it out.
