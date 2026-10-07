---
round: 3
focus:
  - "Webster call sites: for each of the six `webstercli` verbs (`beginbatch.go`, `awaitbatch.go`, `recordbatch.go`, `recoverbatch.go`, `rebaseline.go`, `validate.go`) and `websterengine/runlevel.go`'s `Run`, count the cards in the plan that edit its batch-forming line, and settle whether any card's edit to those lines is wholly overwritten by a later card."
  - "Batcher signature change: settle whether the card that changes the `Batcher` interface's `Batch` signature also moves every caller in the same card, and whether any card leaves callers compiling only through an edit a later card discards."
  - "`RunTally.BaseSHA` in `tools/tokencount`: settle whether the card that adds it names the test that exercises it from a transcript fixture, and whether it states how a `begin-batch` result is recognised among other tool results."
  - "`internal/batcher` tests under `PATTERN-test-economy`: for each test row the profile and config cards add, settle whether another planned row or step already reaches the same behavior through the same public surface."
---

The first entry restates the structural question that decides batching and sequencing of the batcher cards.
The remaining entries cover the test and coverage questions at the new surfaces.
