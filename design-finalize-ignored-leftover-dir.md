# Finalize blocks on untracked leftovers of a project the merge-in deleted

`finalize-ignored-leftover-dir` · Note · bug, landing

Imported from [issue #242](https://github.com/Knatte18/loomyard/issues/242).

## Seen

A darn run's Finalize merged its parent beam into the task worktree after its pull request had merged on GitHub.
The beam had just taken in a main commit that deleted a whole C# project.
The task worktree still held that project's directory with only untracked, gitignored content: `bin/`, `obj/` and generated fixtures.
Once git removed the tracked files, the directory was no longer ignored by the deleted `.gitignore`, so it showed as untracked.
Finalize then blocked: "the task worktree has uncommitted changes after the merge-in: …/NORCE.Import.CommonLibHarness/; commit or remove them on the task branch, then resume".

The run had no live agent left to remove it.
The orch may not edit a task worktree, and the Go driver only asked the parent a question and stopped again on resume.
The only ways forward were a goto back to the writer, which would re-run review and PR rows for a merged pull request, or the operator removing the directory by hand.

## Workaround

The operator removes the leftover directory by hand, then the orch runs `lym loom resume` in the task worktree.

## Expected

Finalize's merge-in treats a directory that holds only files the merge's base ignored, under a path the merge deleted, as leftover output:
it removes it, or ignores it in the clean-tree check, and goes on.
Failing that, the stop names a lym verb the parent can run to clear it, instead of needing a hand edit in the task worktree.
