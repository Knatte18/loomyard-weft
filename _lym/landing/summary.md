# Fix batten cancel hanging on its own pause and failing --remove

`lym batten cancel` could deadlock with a running step, and `lym batten cancel <slug> --remove` failed on records that were already gone.
Both are fixed.

## The cancel hang

The cancel pauses the run, then waits for the in-flight network operations of the run to finish.
A step in Open-IDE held a remote-refs lock and waited at the pause gate for a git call made under that lock.
The cancel waited on the step and the step on the cancel's pause, so only killing the step ended it.

`HoldRemoteRefs` in `internal/gitrepo` now passes the pause gate before it takes the lock.
It stays admitted until the lock is released, so a pause never holds a process that holds the lock.
A pause that starts meanwhile finds the holder's operation marked in flight, and the cancel waits for it as it does for any other in-flight call.

`pauserecord.Admit` is the new primitive.
It passes the gate as `Network` does, marks the operation in flight, and admits the process until its release runs.
`PATTERN-remote-refs-lock` and the `gitrepo` and `pauserecord` package docs state the rule.

## The remove failure

`removeRunRecords` in `internal/battencli` ran `git add -- _lym/shed/<slug>` even when the records directory was gone or empty.
A status-lock `os.MkdirAll` recreates an empty `_lym/shed/<slug>` after the records are removed, so git failed with "pathspec did not match any files".
Every rerun failed the same way, which left the cancel marker in place and made dobby refuse to start the task again.

It now reads whether the records HEAD tracks the run's status or seed, and commits the removal only if it does.
A run directory that is deleted on disk but still tracked still commits its removal, so a removal whose commit failed earlier finishes on a rerun.
A run the records HEAD no longer tracks commits nothing and succeeds.
The tracked check is shared with the cancel's own "records outstanding" probe through `runRecordsTracked`.
A failed read of the records HEAD is returned as an error before any commit or scratch emptying.

## Design decisions

- The admission is process-wide while it stands: a network call on another goroutine or in another repository passes the gate too.
  This favors never holding a lock holder over pausing every concurrent call of the same process during the hold.
  The `Admit` doc comment states it.
- The cancel is not changed.
  The deadlock is removed at the lock holder, so the cancel's wait for in-flight operations stays as it was.

## To check by hand

- Not run: cancel a run during Open-IDE shortly after Seed-IDE and check that `lym batten cancel <slug>` returns without killing the step.
- Not run: `lym batten cancel <slug> --remove` after a plain cancel, and again after a completed `--remove`, and check that both succeed and leave no cancel marker.
