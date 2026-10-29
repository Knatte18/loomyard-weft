# Creel daemon writes no delivery-level log, so an unconfirmed delivery cannot be diagnosed

`creel-delivery-log` · Note · enhancement, creel

Imported from [issue #900](https://github.com/Knatte18/loomyard/issues/900).

While tracing why a `/compact` delivery to the orch session ended as an `unconfirmed` record, the only evidence that the compact had landed was the orch's own Claude transcript.
The creel daemon writes no delivery-level log, so the shuttle error behind the `unconfirmed` record had to be inferred from timing.

This is abnormal because an `unconfirmed` outcome is exactly the case where an operator needs to see what the delivery attempted and what the shuttle returned.
Without a log, a wrong suspected cause (here, a race with the `asyncRewake` Stop waiter) survived until the transcript was checked by hand.

What to do: have the creel daemon log each delivery attempt, with the target session, the shuttle result or error, and the final confirmed or unconfirmed decision, at a path an operator can find from the `unconfirmed` record.

Provenance: task `compact-inject-draft`, note `Darn-Write.md`.
