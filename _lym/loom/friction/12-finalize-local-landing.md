# Implementing the local landing

A parent ahead of origin by owed landing tips always reads as moved to Finalize's locked pass.
The lock-free pass merges and records as `gatedTip` the upstream tip (`parentMergeTip`), while the parent's head is past it,
so every such landing loses each of its `landing.finalize_lost_races` races and then merges and gates under the parent lock.
The landing is correct, but it holds the lock across the verify for as long as an earlier landing's push is owed.
Neither the card nor its targets cover `gateUnlocked` or `parentMergeTip`, so it is left as it is.

An unreachable origin in a test has to be a refused connection (`http://127.0.0.1:1/x`):
a missing directory makes the origin reconcile return an error instead of reporting origin unreachable.
