# Teardown removes the whole worktree directory, and a gone pair blocks no board edit

`teardown-gone-pair` · Task · bug, batten, fabric, board · running · Drift-Verify

Two faults from issues #54 and #52, seen in hub Models-LYMHUB, beam `duplication-and-import-followups`, 2026-11-04. Fix both.

## 1. Worktree-Teardown leaves the worktree directory behind (#54)

The motif task `knut-rheology-friction-pvt` ran to the end: Run-Shed, Land and Worktree-Teardown done.
The directory `Models-LYMHUB/knut-rheology-friction-pvt` was left behind: no git worktree any more, holding only `obj/` and `bin/` build output.
The same leftover exists for many earlier runs in that hub.

Expected: Worktree-Teardown, and `lym fabric remove`, removes the whole directory, ignored build output included, and fails loudly if it cannot.

## 2. A gone pair still blocks a board edit, and cancel cannot clear it (#52)

After the run was done and `lym fabric pairs` listed no pair for the slug:

```
lym board upsert '{"slug":"knut-rheology-friction-pvt","project":"knut-rheology"}'
-> entry has a pair, ... end the pair first with lym batten cancel knut-rheology-friction-pvt --remove, then change project
lym batten cancel knut-rheology-friction-pvt --remove
-> battencli: could not read the task worktree of knut-rheology-friction-pvt: not a git repository
```

This held after `lym batten cleanup --apply`; only the operator's manual `rm -rf` of the leftover directory unblocked the board write, and the beam could not be emptied for its landing meanwhile.

Expected: the board's "has a pair" check and `lym batten cancel --remove` treat a directory that is no git worktree as no pair, or clear the record when the worktree is already gone; a finished run whose pair is torn down leaves nothing that blocks a board edit.
