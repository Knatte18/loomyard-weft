# Name the cause when a shed loop ends without an envelope

A shed loop that ended without an envelope halted the step with `loop exit: not recorded, since no process marked the loop's exit`.
The note could not tell a kill, a machine stop or a lym defect apart.
Before such an end, the shuttle logged `session state disagrees ... loop=held state=busy cause=background` for a state that is expected.
This change makes every such end carry a cause and stops the warning for that pairing.

Closes #150

## What changes

- `internal/shedverbs/loop.go`: a loop that ends before its first step marks the reason in the loop log through `endWithoutEnvelope`.
  The ends are a start refusal, a loop lock another loop holds, and a directory or pid file it cannot write.
  A loop that cannot write its envelope marks that too.
  The waiter's dead-loop stop reads these marks as the loop's exit record.
- `internal/proc`: new `BootID`, read from `/proc/sys/kernel/random/boot_id` on Linux and always false on Windows.
  The pid file records it as `boot_id` (`LoopPIDRecord.BootID`), and `LoopSpec.BootID` overrides the read.
- `internal/shedverbs/loopwait.go`: for an exit no process marked, the dead-loop stop compares the recorded boot id with the current one.
  A different id names a stop of the machine as likely.
  The same id names a kill of the loop and its keeper that no process can catch as likely.
  An unknown id names both, as the note did before.
- `internal/shuttleengine/waitshadow.go`: a held loop beside a session `busy` on background work is now an expected pairing and logs nothing.
  A turn end counted held leaves shells running that the loop no longer waits on, as it does for done.
- `LoopSpec.LockGrace` makes the lock wait settable, so a test need not wait the default few seconds.

## Design decisions

- The boot id decides between a machine stop and a kill, not the wall clock, which a clock step moves.
  The cause is worded as likely, since a kill followed by a boot cannot be told from a boot that killed the loop.
- The mark about steps the loop reconciled at its start records no exit.
  It no longer hides an unmarked exit; it is listed apart as `loop marks`.
- The third item of the issue, escalating a repeating warning, is met by removing the warning for the expected held-beside-background pairing.
  Other disagreements still log once per change, as before.

## To check

- `reconciliation-mark-hides-unmarked-exit` and `unmarked-exit-cause-wallclock-boot-time` were open findings in `internal/shedverbs/loopwait.go`.
  The diff addresses both through the `loop marks` split and the boot id comparison; confirm they hold.
- A pid file written before this change holds no boot id, so its stop reads as unknown and names both causes.
  Check that this wording is acceptable for such a file.
- The change was not run against a real kill or machine stop; only unit and integration tests cover the causes.
