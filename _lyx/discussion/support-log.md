# Support log — loop-exit-halt-names-its-cause

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Carry the loop process's exit status from the waiter's spawn into the cause: only a waiter that spawned the loop sees it, and the `LoopSpec.Spawn` seam's channel carries no value, so every fake spawn would change for a reading most dead-loop stops lack.
- Install a signal handler in the loop that records the signal in the pid record: platform-specific, it cannot catch a hard kill, and it changes how the loop dies.
- Rename the envelope's `(loop-exited)` token to the richer cause: drivers and tests match the token, and the halt note is where a reader looks for the cause.
- A separate anomaly kind per interrupt cause (signal, watchdog, status read): the cause text already names it, and the classifier reads no cause.

## Review rounds

_No rounds yet._

## Question ledger

- Should the watchdog and status-read interrupted stops take the new kind too? Self-pick: yes, they share the interrupt hook and none is a producer failure.
- What wire spelling? Self-pick: `interrupted-stop`, after the loop's `interrupted` stop kind.
- Which trace file when the in-flight step's trace is unknown? Self-pick: the loop log, as before.
