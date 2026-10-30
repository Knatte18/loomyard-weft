---
round: 1
focus:
  - "internal/loomcli/wiring.go around the `Cwd` assignment and the pawl branch of wire: does a pawl run's Env carry the absolute `Cwd` the `Preflight` registry row's `requireAbsRoot` demands, so the recipe builds for a pawl seed and not only in the tier-0 recipe test?"
  - "pawl-recipe.yaml, the recipe's header comment and the new `Preflight carries no on_stuck` paragraph: does the comment's reason fit this row, and does the recipe's own on_stuck convention for the other rows still read correctly?"
  - "Pawl-run docs and comments that list where a pawl run stands before Parent-Align (docs/overview.md, internal/pawlengine/doc.go, internal/battenshed/doc.go, internal/orchcli/doc.go, internal/beamcli, internal/battencli): does any still spell the old first rows, or the hold-free rows, without Preflight, as the decision record's scope item on docs asks?"
  - "boardengine.RunHolds and every reader of `holdFreeProducers`, including the dobby slot rule (`runTakesNoSlot`) and the `lym loom peers` list: does adding `Preflight` change behavior for a non-beam entry or for a halted run at Preflight in a way the record's D2 does not bound?"
  - "Test coverage of the new row: besides the table rows added, which behavior the diff adds (Preflight routing to Beam-Up, no on_stuck, hold-free at Preflight) has no pinning test, and does any changed test name or assertion duplicate an existing test?"
---

Seed rationale: the change is small and mostly recipe, docs and tables.
The main risk is a seam outside the diff that the new row needs, chiefly `Env.Cwd` for a pawl wiring, and stale doc listings the diff did not reach.
