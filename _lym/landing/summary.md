# Retry pair-create origin reads; treat a not-yet-made pair as fresh

A batten run blocked at Worktree-Create when one `git ls-remote` of the weft branch got a connection reset,
although the same lookup succeeded seconds later.
The stop question then claimed the task worktree had already been created,
and asked whether to restore the absent weft branch on origin or abandon the pair.
Nothing had been made.

Closes #279

## What changes

- Some of the create's origin reads now run again after 1, 2 and 4 seconds when they fail in transit:
  `Add`'s warp and weft pre-flight probes, the fetch of an adopted warp branch,
  the fetches of a live weft branch, the lookup of a weft branch that is only on origin,
  and Worktree-Create's lookup of a beam's branch.
  A failure that is not a transport failure is returned at once.
- Before Worktree-Create has made the pair, `primeShuttle` no longer tries to locate the task worktree.
  It therefore no longer logs a warning whose error says the run's status records the worktree as created.
- The run descriptions given to the tackler and the arbiter now say that,
  until Worktree-Create has finished done, an absent task worktree and a records branch absent from origin
  are the fresh state, never a pair to restore.

## Design decisions

- The retry is `AddOptions.RetryOriginReadInTransit`, and it waits through `AddOptions.BackoffSleep`.
  It is exported so that batten's beam lookup can use it, and it is meant only for reads that are safe to repeat.
- `gitexec.IsTransportFailure` decides what is retried, and it counts a timed-out git command.
  The retries inside `Add` can therefore hold the prime lock longer:
  each retried read adds up to 7 seconds of backoff and three more attempts,
  and each remote command in an attempt can run to git's remote deadline.
- Two lookups in `Add` still run once: the pair's branch name under `AdoptOnly`,
  and the warp's default branch under `AdoptLocal`.
- `resolveWeftBranch` now takes the reader as a parameter, so checkout and reconcile still read once.
- The wrong stop question is addressed in two ways: the misleading warning is gone, and the agents' prompt text is explicit.
  No code constrains what question an agent asks.

## To check by hand

- `primeShuttle` (`internal/battencli/drive.go`) is only partly tested.
  `TestPrimeShuttle_WiresTheReplyProbe` (`internal/battencli/wire_seams_test.go`) fails if the absent-pair path logs
  a failure to locate the task worktree,
  but no test checks that the usage recorder is wired when the pair is present.
- No test checks the new tackler and arbiter wording against a live agent's stop question.
- The retry test drives the pre-flight probes.
  No new test makes the later fetches, the origin-only weft lookup or the beam lookup fail in transit.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
