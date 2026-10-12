# Plan does not surface tests that pin a method set a card extends

`plan-pinned-tests` · Note · bug, loom

Imported from [issue #594](https://github.com/Knatte18/loomyard/issues/594).

**What happened.** A card added a method to the `r.runChecked` method set.
`cmd/lyx/gitrepoboundary_test.go` pins that set by name, so the change breaks it.
The plan named no card edit for that file, and the break surfaced only when the agent ran `./cmd/lyx` by hand.

**Why abnormal.** A card's declared `Edit:` list and gate steps did not include a test the change necessarily affects, so the per-card gate stayed green while a cross-package boundary test failed.

**What to do.** Have plan or card validation flag cards that change a symbol pinned by name in a boundary test, or run the boundary-test package in the card gate so the break is caught at the card that causes it.

Provenance: task `convergence-plan`; note `06-board-and-archive-readers.md`.
