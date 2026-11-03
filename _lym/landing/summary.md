# batten cancel --remove: no leftover directory blocks the next run

`lym batten cancel <slug> --remove` could answer `pair_removed: true` and still leave a plain directory at the task worktree's path.
A late writer, such as a dotnet build server or the reed session's exit hooks, refilled it with `obj/` output and `.lym/reed.lock` after the removal.
That directory is no worktree, so the next `lym batten run <slug>` failed at Worktree-Create with `not a git repository`, `lym fabric prune` listed nothing, and `lym board rename` refused because it saw a pair.
This change closes each of those.

Closes #39

## What changes

- `fabricengine.Remove` looks at the task worktree's path once more after the sibling's teardown and removes a directory a late writer refilled there.
  One it cannot remove is logged and reported in `RemoveResult.StrayPath`; the removal still succeeds, since the pair is gone.
- `batten cancel --remove` turns a `StrayPath` into a warning in its report, naming the directory and saying to save anything worth keeping and delete it before the slug runs again.
- Worktree-Create (`battencli`) refuses a directory at its path that holds no registered worktree with a precondition naming the path and the way forward: save what is worth keeping, delete the directory, run again.
  A path that does hold a registered worktree that fails to resolve gets the remove-the-pair remedy instead.
- `fabric prune` reports every hub directory named as a valid pair slug that no worktree registration covers, in both dry-run and `--apply`, as `Unowned` with the way to clear it in `Error`.
  Hidden directories and slugs with a live warp are skipped.
- `board rename` and project change test the pair with the new `fabricengine.PairWorktreeStands`, so a leftover directory no longer reads as a pair.
- The `ErrTaskPathTaken` message now points at `lym fabric prune`, which reports such a directory, instead of saying fabric cannot see it.

## Design decisions

- Prune never removes a stray directory.
  Nothing proves fabric made it, so it only reports.
  `Remove` deletes the refilled directory because its own removal of the registered worktree is the ownership proof.
- `PairWorktreeStands` reads true when the path is present but the repository's worktrees cannot be listed.
  The answer guards refusals, so an unlistable repository must keep refusing.
- The cancel gets no wait on the processes it ends.
  The re-check after teardown removes whatever they wrote, and a refill after that last look is covered by the Worktree-Create refusal and the prune report.

## To check by hand

- The `Unowned` pass in `Prune` (`internal/fabricengine/prune.go`) has no tests for its negative rows: hidden directories, directories with a live warp slug and names that are no valid slug are skipped, but nothing pins that.
- The failed-delete branch of the post-teardown re-check in `Remove` (`internal/fabricengine/remove.go`), where the refilled directory cannot be deleted and `StrayPath` is set, has no test.
- Run `lym batten cancel <slug> --remove` against a real task whose build server writes `obj/` late, then `lym batten run <slug>`, to confirm the end-to-end case from the issue.
