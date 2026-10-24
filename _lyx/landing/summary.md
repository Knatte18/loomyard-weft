# gitexec: run every git child with GIT_OPTIONAL_LOCKS=0

Every git child spawned through `internal/gitexec` now runs with `GIT_OPTIONAL_LOCKS=0`, local and remote alike, whatever the process environment sets it to.
A read-only probe such as `git status --porcelain` no longer takes `index.lock` to write back a refreshed stat cache.

Why: lyx's locked commits serialize on the weft write lock, but its read-only probes run without it.
A probe left running from an earlier `lyx loom start` could hold `index.lock` at the instant the next start's `git add` ran, which failed with `index.lock: File exists`.
That made `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` flake and halted a run at Merge-In.
With optional locks off, the only `index.lock` holders are lyx's own locked writers.

Closes #856

## Design decisions

- The fix is in `gitexec`, not in the smoke test.
  The race is between any two lyx processes on one weft, so a wait between starts in one test would hide it there and leave the cause in place.
- `childEnv` drops any inherited `GIT_OPTIONAL_LOCKS` entry and appends `GIT_OPTIONAL_LOCKS=0`, so the child sees exactly one value and an operator's setting cannot re-enable the lock.
- Bound: a read probe no longer refreshes the stat cache, so a later probe may re-read files a refresh would have cached.
- Git spawns outside `gitexec`, such as a test helper's raw `exec.Command`, keep git's default optional locks.

The `internal/gitexec` package doc states the new environment entry.

## Tests

`TestRun` gains a row that sets `GIT_OPTIONAL_LOCKS=1` and asserts the child environment holds `0` alone.
The remote-bounds row also pins `GIT_OPTIONAL_LOCKS=0` in a remote command's environment.

## To check by hand

The flake is a load-dependent race and was not reproduced, so no run shows it gone.
Watch tier-2 gate runs for a recurrence of `index.lock: File exists` in `internal/loomcli`.
