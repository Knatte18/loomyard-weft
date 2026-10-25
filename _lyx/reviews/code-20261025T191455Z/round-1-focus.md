---
round: 1
focus:
  - "internal/lyxcwd/anchor.go and lyxcwd.go: does the own-marker read and write follow card 01 and the decision record, and do callers treat a missing or invalid marker as the record says?"
  - "internal/fabricengine anchor-change engine (cards 13, 15) and internal/hubforge/reanchor.go: is every failure path mid-move recoverable, and do the weft writers stop while a move is recorded?"
  - "Refusal guards (cards 09-12, 19): does each of weft pull and push, checkout, drop-local, origin reconcile, promote and adopt refuse in every case the record names, and none beyond it?"
  - "internal/hubgeom/dobbygeom.go and the dobby run listing (card 16): does a pair with an invalid anchor marker or a move in progress behave as the record says?"
  - "Docs landing in the same diff: doc.go of each touched package, docs/overview.md, and the PATTERN.md and pattern/ changes the record declares."
  - "Test economy: new tests that duplicate existing coverage, and behavior added with no test."
---

Seed focus from the diff stat and the plan's card list.
