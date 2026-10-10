---
round: 3
focus:
  - "Recording of the seed drop on the push path: card 13 makes `Bolt.Push` pass `dropSeedCommits` a local `Mutations`, and cards 10 and 11 edit callers (`hubreconcile`'s push path, `runReconcile` in `internal/fabriccli/fabric.go`) that reach `Bolt.Push`. Settle whether each caller that emits a mutation envelope carries the drop in that envelope, as `PATTERN-mutation-record` requires of a mutating fabric verb, and whether card 13's Intent states which callers record and which only log."
  - "Card 7 test write surface: its tests sit in a `package fabricengine_test` file yet drive unexported `dropSeedCommits` and `boardDropRequest`. Settle whether every file the tests need, including the package's `export_test.go` shim, is on card 7's `Edit:` or `Create:` list, and whether each fixture, the foreign `_board` one included, states its construction through `hubforge`."
  - "Card 8's scenario test: it passes a fixed identity to `seedStencilsAt` and asserts each seed subject carries a label. Settle which source the subject's label comes from, `BinaryLabel()` of the test process or the passed identity, and whether the card states the same source the assertion uses."
  - "Cherry-pick replay in cards 5 and 7: a non-seed ahead commit whose change upstream already carries. Settle whether the plan states the outcome, skipped or reported as a conflict, and whether a test row reaches it."
---

Round 3 focuses on the one gating finding that remains and the four small gaps beside it.
