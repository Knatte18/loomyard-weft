# batten cancel --remove leaves a non-git directory that blocks Worktree-Create

`cancel-remove-leftover-dir` · Task · bug, batten, fabric · running · PR-Review

Imported from [issue #39](https://github.com/Knatte18/loomyard/issues/39).

## What happened

In the Models hub (anchor `src/csharp/NORCE.Models`), `lym batten cancel production-model-duplicates --remove` stopped a tapestry run in Discussion-Write and answered `pair_removed: true`.
The task worktree directory `<hub>/production-model-duplicates/` stayed behind, no longer a git worktree, holding only `src/csharp/NORCE.Models/*/obj/` build output and `.lym/reed.lock` (52 files, all timestamped at the cancel).
A later `lym batten run production-model-duplicates` failed at Worktree-Create with `{"error":"not a git repository"}`, and the drive report handed back as an interrupted step.
`lym fabric prune` listed nothing, since the directory is not a worktree.
Removing the directory by hand let the run start.

`lym board rename` of the same slug after the cancel also refused with "has a pair, and its slug names that pair: end the pair first with `lym batten cancel ... --remove`", although that cancel had already run.

## Expected

- `batten cancel --remove` leaves no directory at the pair's path, or waits for the processes it ends (a dotnet build server or the reed session likely wrote `obj/` and `reed.lock` after the removal) before removing it.
- Worktree-Create meets a leftover non-git directory at its path with a clear refusal naming the path and the way forward, or clears one holding only ignored build output.
- `fabric prune` reports such an orphan directory.
- `board rename` does not see a pair once cancel has removed it.
