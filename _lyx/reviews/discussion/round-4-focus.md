---
round: 4
focus:
  - "D7 in decision-record.md, the refusal when the remote task branch holds commits the local HEAD lacks: settle whether the stated way forward converges. After the operator follows it and re-runs `webster reset`, does the reset's remote update drop the commits the way forward just integrated, and does the Decision say who resolves a merge conflict and how? Check the Bound against a merge that makes the listed commits reachable from the local HEAD, and check the way forward for the case where the listed commits may be someone else's work."
  - "D7 in decision-record.md, order and environment of refusals: settle which refusal the operator sees first when the remote-ancestry refusal and a local gate refusal (a dirty tracked path) both apply, and what `webster reset` does with no remote configured and with an unreachable remote. Check that each distinct refusal named has a refusal-spec row requirement."
  - "D4 and D6 in decision-record.md against `BatchFailedError.Error` in `internal/websterengine/batchfail.go` and `RecoverSpawnOrAttach`, which refuses a failed batch whose record lists uncheckable paths with `run --fresh`: settle whether the way forward named in the batch-failed reason and by `rebaseline --card` works the first time for a failed batch carrying uncheckable paths."
  - "D6 in decision-record.md against `awaitTerminal` and `recoverSpawn` in `internal/websterengine/recoverbatch.go`: settle whether the Decision states what the restamp of a dead batch lets through while its strand may still work on the old card, what bounds that window, and which step ends it."
  - "D10 in decision-record.md against `BeginBatch`, `State.AssertedModel` and the `Run` entry in `internal/websterengine/runlevel.go`: settle whether the rationale and Bound describe what the code compares and when it can diverge in the shipped flow, and whether the list of what is removed matches every write of `AssertedModel`."
  - "D1 in decision-record.md against `planglyph.ValidateDispatch` and `deriveEdges`: settle whether the Decision states which card a pure-format finding is attributed to, so the dispatch scoping to unbegun cards is decidable for a pair with one begun and one unbegun card, and whether the card-order claim states the contiguous-batch assumption it rests on."
---

Round 3 left D7, D6, D10 and D1 open; the entries above name the questions to settle there.
Acceptance-criteria coverage for D6, D7 and D10 follows from how those Decisions settle.
