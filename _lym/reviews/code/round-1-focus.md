---
round: 1
focus:
  - "internal/loomcli/darnwiring.go and internal/loomcli/gatefixer.go: does `writerFixerOn(c.recipe, darnWriteEngine, darnGateEntry, ...)` use the same condition the Darn-Write row gate uses to attach a fixer (`gateFixerSeam`), including the pair-slug check and the recipe argument? Settle whether the prompt and the gate can disagree."
  - "internal/darnengine/spec.go `makeChangeList`: the fragments are read at call time through `stencilstore.Read` from `stencilsDir`. Settle that both new fragment names resolve in every place the darn stencil resolves (embedded registry, any stencil-install or override path), and that a missing fragment fails with a usable error."
  - "contracts/stencils/darn/darn-fragment-make-change.md against the removed Step 4 text in darn-template-write.md: settle that the ungated rendering is byte-identical in content to the old list, as the record requires (`TrimSpace`, indentation, blank lines around the marker)."
  - "contracts/stencils/darn/darn-fragment-make-change-gated.md: settle that no instruction left in the gated rendering, or elsewhere in darn-template-write.md, still tells the writer or its subagents to run `lym gate test` or leave commits green by a gate run; and that the Plan-Write gated-fragment pattern (marker, comment header, wording of acting only on what the fixer leaves) is followed."
  - "darn-template-write.md header comment and the `subagent_model` marker: settle that moving `subagent_model` from the writer stencil into the fragments leaves no unfilled or stale marker in either stencil, and that marker tests (contracts/stencils/markers_test.go) and the engine's required-marker handling agree."
  - "internal/darnengine/spec_test.go: settle that both renderings are pinned as the record asks, and judge the new assertions against test economy (overlap between the two table rows, assertions on static prose)."
  - "Other callers of `darnengine.DarnTable` / `DarnInputs` and of `wireDarn`: settle that every call site passes the new argument or field consistently, and that docs named for this kind of change (darnengine/doc.go, loomcli/doc.go, any PATTERN or overview entry) land in the diff."
  - "_lym/landing/summary.md: check each claim against the diff."
---

Seed focus for round 1. The answer key is the decision record alone; its `## Goal` holds the board entry.
