# Some test writes a .git into the shared temp directory and fails unrelated packages

`stray-tmp-git-writer` · Task · bug, gate, tools · starting · Worktree-Create

## What was seen

On 2026-11-18 the Finalize fixer of `board-view-not-refreshing` met a red verify: `shedverbs` (tier 0) and `shedcli` (tier 1) failed outside any test,
only on `tmuxkit`'s check for a `.git` that appeared directly in the shared temp directory (`/tmp/.git`); no test failed.
The task's diff touched only `internal/boardengine`, and a rerun passed clean.
The fixer reports the same `/tmp/.git` finding in `_bolt` gate runs of unrelated packages since 2026-11-08.

## Why it matters

The check in `internal/testkit/tmuxkit/tmuxkit.go` is machine-wide: it cannot tell which test binary wrote the entry,
so it fails every package that runs at the same time, and each run's fixer spends a round on a failure its task did not cause.
The writer itself is a real bug: a test or the code it drives writes outside its own `t.TempDir`, often through a path joined onto an empty directory.

## Idea

1. Find the writer: record which packages were running when the entry appeared (the gate slots know), or watch the temp root for the create and name the process,
   and fix that test.
2. Make the check blame only the package that wrote the entry, or report it as a machine finding that does not fail a package whose own run did not create it.
