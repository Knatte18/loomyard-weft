# Teardown deletes a stray worktree directory, or fails loudly

A directory that is no worktree, left at a task worktree's path after its pair is gone, now gets deleted by the routes that own the slug, or fails them loudly.
Before, `lym fabric remove` and batten's Worktree-Teardown reported success when a late writer, such as a build or language server, refilled the directory with `obj/` and `bin/` output and the final delete failed.
Nothing then cleared it: `lym batten cancel --remove` refused with "nothing left", and `lym batten cleanup` and `lym fabric prune` only reported it.

Closes #54
Closes #52

## What changes

- `fabricengine.Remove` finishes the teardown as before when the post-teardown re-check cannot delete a refilled directory.
  It then returns an error wrapping the new `ErrStrayPathKept`, naming the directory and the way forward, instead of a success with only `StrayPath` set.
  The respawned pair push that follows a failed removal is skipped for this error, since no worktree is left to push.
- New `Topology.RemoveStrayTaskPath` deletes what stands at a slug's task worktree path when it is no registered linked worktree.
  It refuses a registered worktree and a directory holding a `.git` entry.
  The delete goes through the existing contained `removeWorktreeLeftover` with its late-writer retries; a failure wraps `ErrStrayPathKept`.
- `PairParts` gains `StrayPath`, read from the same `inspectPair` that `Remove` uses, and `Any()` counts it.
- `pairteardown.Request` gains `ClearStray`.
  Once the removal left the pair gone (success, `ErrPairNotFound` or `ErrStrayPathKept`), `RemovePair` calls `RemoveStrayTaskPath` before it settles the board claim.
  A failed delete is returned.
  A delete that clears a kept directory turns `ErrStrayPathKept` into success.
- Batten's Worktree-Teardown sets `ClearStray`.
  This second look runs after `Remove` has returned and regenerated the hub workspace file, which drops the pair's folder from an open editor.
  A delete that still fails stops the row with "once it is deleted, resume this run with `lym batten run <slug>`".
  The resumed row then deletes the directory, or finds it gone.
- `lym batten cancel --remove` sets `ClearStray` and drops its old warning about a stray path.
  With only a stray directory left, cancel without `--remove` refuses and names `--remove`.
  With `--remove`, cancel deletes the directory instead of refusing with "nothing left".
- The beam-land member listing reads a member's pair through `PairWorktreeStands`, so a leftover directory no longer reads as a pair in the landing refusal's way forward.
- The fabricengine and pairteardown package docs and `PATTERN-pair-teardown`'s background file describe the new behavior.

## Design decisions

- Fault 2's two named symptoms were already fixed on main by 56a77b49 and 5762ac04, which landed the day before the report.
  The board's "has a pair" check uses `PairWorktreeStands`, and `taskWorktreePresent` reads a non-worktree directory as absent.
  The hub that reported them most likely ran an older binary.
  This change closes the rest of #52: the leftover directory is now deleted by the cancel the refusal names.
- The ownership proof for deleting a non-worktree directory stays with the caller.
  Batten's own run created the pair at that path, and `cancel --remove` is the operator's explicit removal of that slug's pair.
  `fabric prune` and `fabric remove` on a stray-only slug still never delete one, as 5762ac04 decided.
- `fabric remove` does not set `ClearStray`: its `Run` stops at the refusal probe's `ErrPairNotFound` before `RemovePair`.
  Its way forward for a kept directory is to delete it by hand.
- No settle delay was added before the second look.
  Without one, a writer that refills the directory after the teardown has fully returned can still leave it behind.
  That directory no longer blocks a board edit or a beam landing, and `cancel --remove` now clears it.

## To check by hand

- Not run end to end on a real hub: a C# pair whose language server writes `obj/` after the teardown, taken through Worktree-Teardown and then `lym batten cancel <slug> --remove`.
- The `ErrStrayPathKept` path of `Remove` under `RemoveLocal` (local mode) has no test of its own.
