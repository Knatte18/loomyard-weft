# Run every gitexec git child with GIT_OPTIONAL_LOCKS=0

`TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` failed once at Merge-In when its second `lyx loom start` got `index.lock: File exists` on the pair's weft worktree, and the run halted for a human (issue #856).

## Cause

Every records commit takes the weft write lock, so two locked writers never meet on the index.
Read-only probes such as `git status --porcelain` run without that lock, and git's `status` takes `index.lock` opportunistically to write back a refreshed stat cache.
The first start's driver and origin reconcile probe the weft this way, so one probe can hold the index lock at the instant the second start's records commit runs `git add`.
The race is in production, between any two lyx processes on one weft, not only in the test.

## Change

`internal/gitexec`'s child environment now sets `GIT_OPTIONAL_LOCKS=0` for every git child, local and remote, and replaces any value the process environment carries.
With optional locks off, a read probe never touches the index, so only lyx's own locked writers take `index.lock`.
The package doc's Environment section states the entry and its bound: a read probe no longer refreshes the stat cache, so a later probe may re-read files a refresh would have cached.

`TestRun` gains a row that sets `GIT_OPTIONAL_LOCKS=1` and asserts the child's environment holds the one value `0`.

## Design decisions

- The fix lives in `gitexec` rather than in the smoke test: a `waitRecordsIndexLockGone` between starts narrows the window in one test only, while the first start's driver keeps probing after the wait returns.
- No retry on `index.lock` at the start's commit site: that would protect one writer and leave every other locked writer exposed to the same probe.
- The smoke test is unchanged.

## Bounds

A git spawn outside `gitexec`, such as a test helper's raw `exec.Command`, keeps git's default optional locks.
