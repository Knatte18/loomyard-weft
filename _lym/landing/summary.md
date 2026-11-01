# batten cancel: no hang behind a paused step; --remove is idempotent

Two fixes for `lym batten cancel`.

## A remote-refs lock holder passes the pause gate before the lock

The hang came from the remote-refs lock, not from a pause the cancel wrote.
The cancel writes no pause.
The run's traces show that the operator's HALT button (`lym tender pause plan-spec-split`) wrote the pause at 19:33:49 and CANCEL ran at 19:34:14.
The running `lym batten step` (Open-IDE) then pushed through `gitrepo`, which takes the repository's remote-refs lock and runs git under it.
`gitexec` waits at the pause gate inside that call, so the step waited for the pause to lift while it held the lock.
The cancel is exempt from the gate, but its origin reconcile takes the same lock (`HoldRemoteRefs`) with no bound.
It logged nothing until the step was killed at 19:36:02, and it finished a second later.

- `pauserecord.Admit(op)` passes the gate as `Network` does and marks `op` in flight.
  It then admits the process until its release runs.
  While an admission stands, every network call of the process passes the gate at once and is still marked in flight.
- `gitrepo.Repo.HoldRemoteRefs`, which every fetch, pull and push method and the fabric reconcile go through, takes the admission before the lock and drops it after the release.
  A process that waits at the gate holds no lock, and a lock holder is never held by a pause.
  A pause that starts while a holder runs sees its calls in flight, as it sees any call already past the gate.
- `PATTERN-remote-refs-lock` now states that the pause gate is passed before the lock is taken.

With this fix the cancel's reconcile goes through, and its end of the run's registered processes reaches the paused loop and its step.

## cancel --remove after a plain cancel succeeds

`removeRunRecords` committed the run directory's removal in every case.
After a plain cancel had already removed and committed the records, `lym batten cancel <slug> --remove` ran `git add -- _lym/shed/<slug>` on a path that was neither on disk nor tracked.
That failed with "pathspec did not match any files" and left the cancel marker, which refuses the slug's next start, and every rerun failed the same way.
The removal now commits only while the records HEAD holds the run's status or seed.
A removal that already finished succeeds again with no commit, and a deleted directory the HEAD still tracks still commits, as before.
`runRecordsTracked` is the one reader of that question, shared with the cancel's existing records probe.

## Design decisions

- The fix sits at the lock, not in the cancel.
  The decision record's idea was that a cancel "ends the step or does not pause before it".
  The cancel never paused, and every process that wants the remote-refs lock had the same exposure behind a paused holder, a dobby pass or a cleanup included.
  The cancel's refuse-before-ending order stays as it is.
- The admission is process-wide, not per call.
  The fabric reconcile holds two repositories' locks in turn and runs git through both `gitrepo` and `gitexec`, so a per-call flag would not reach every call under the lock.
  While a process is admitted, its other network calls also pass, and they are marked in flight, so a pause reads that process as busy.
- The decision record traced the `--remove` failure to `os.MkdirAll` recreating `_lym/shed/<slug>`.
  That call creates the status lock's directory under `.lym`, not `_lym`, so it is left alone.
  The failing `git add` is explained by the commit running on a path that was neither on disk nor tracked.

## Tests

- `TestNetwork` gains a row: an admission passes a later call at once with both marked in flight, and once the admission is released a call is held again.
- `TestHoldRemoteRefs_PassesThePauseGateBeforeTheLockAndStaysAdmittedUntilTheRelease` (tier 1, real git) checks three things.
  The lock is free while `HoldRemoteRefs` waits at a live pause.
  A call made while the lock is held passes a new pause.
  A call made after the release is held again.
- `TestRemoveRunRecords_CommitsTheRemovalOnlyWhileTheRecordsHeadTracksTheRun` covers four states of the run directory: tracked on disk, deleted but tracked, gone and untracked (the regression), and untracked on disk.
  In each, the directory is removed and the scratch is emptied.
- The cleanup fixture now records a committed seed for each run directory it writes, as a real run has.
