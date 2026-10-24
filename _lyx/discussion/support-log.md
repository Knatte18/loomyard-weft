# Support log — drop-transitional-tag-spellings

## Interview

A darn run holds no interview: the writer read the board entry and the code and made its own calls.

## Rejected alternatives

- Letting the one-tag scan ignore old spellings once its allowance is gone: a file still tagged `integration` would then pass the scan while no tier command selects it, so its tests silently stop running.
- Leaving `cmd/lyx/llmtier_test.go`'s `llm` allowance because the board entry does not list it: the guard would accept a spelling the one-tag scan refuses.
- Hand-editing `_board/_lyx/config/gate.yaml`: `lyx config` is the file's writer and commits it on the board.
- Sweeping every comment that mentions `-tags integration`: out of the board entry's scope and noise for review.

## Review rounds

_No rounds yet._

## Question ledger

- Has the deploy the entry waits on happened? Self-picked yes: the hub's `gate.yaml` was last reconciled by the production build of the commit that brought the tier tags.
- Should the one-tag scan refuse old spellings or ignore them? Self-picked refuse.
- Does the LLM tier guard's `llm` allowance go too? Self-picked yes.
- How is the hub's `gate.yaml` changed? Self-picked `lyx config gate --set`.
