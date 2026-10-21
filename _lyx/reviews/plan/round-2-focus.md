---
round: 2
focus:
  - "Card 9 (`_lyx/plan/09-reconcile-failure-warns.md`): where its tests capture the process-wide logger via `internal/testkit/logcapture`, does any capturing test run under a `t.Parallel` parent, or alongside parallel tests in `internal/hubreconcile`, such that another test's Warn can land in the buffer? Settle whether each capture sits in a top-level non-parallel test."
  - "Card 9's loom resume row in `smoke_prebootstrap_integration_test.go`: the built binary's combined output carries the Warn that `EnsureOrWarn` prints to stderr. Does every output assertion the card states hold against a correct implementation, and does the card claim any refusal text the loom config load does not produce?"
  - "Card 6 (`_lyx/plan/06-sparse-hub-base-reconcile.md`): on a present base file, which function drops a template-equal per-name entry and reports it in `Result.Dropped`? Settle that the named mechanism can produce that drop, given what `yamlengine.Prune` and `yamlengine.Reconcile` each return."
  - "Card 8 (`_lyx/plan/08-walk-strips-base-once.md`): for each member whose parameters change so a hook reaches it, does the Edit glyph carry the re-sign arrow, and is every caller that must pass the new parameter a target?"
  - "Card 11 (`_lyx/plan/11-clone-writes-base-marker.md`): after the marker write, what state does `CloneAndWire` leave if its board commit fails, and does the next start verb's walk recover it as card 8 describes?"
---

Focus names where to look; it states no verdict and caps no severity.
