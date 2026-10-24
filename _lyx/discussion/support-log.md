# Support log — webster-reset-reopens-dead-batch

## Interview

A darn run holds no interview: the writer read the board entry and the code and made its own calls.

## Rejected alternatives

- Make the reset set the batch non-terminal: `RecoverSpawnOrAttach` attaches to a non-terminal recovery record's strand, which is dead, instead of spawning a fresh recovery.
- Make the reset rewrite the batch's status to `stuck` or `failed`: the status mirrors the batch's digest and report, and `failed` carries suspect-path and batch-failed meaning the reset does not create.
- Add a reset marker field to `BatchState` that `RecoveryRetry` reads: the cleared count already marks the state, and a second field would have to stay in step with it.
- Teach the Master stencil to run `recover-batch` after any reset: Merriam does not see that a reset ran, and the verdict would disagree with `status` and `begin-batch`.

## Review rounds

_No rounds yet._

## Question ledger

- Which of the board entry's two expected outcomes to build? Self-pick: the verdict reflects the cleared count and Merriam runs `recover-batch`; resolved.
- Should a dead recovery that counted nothing because it rendered an amendment also retry? Self-pick: yes, the uncounted-spawn rule already allows its re-run and the next spawn is counted; resolved.
