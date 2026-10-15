---
verdict: APPROVED
---

No finding: the merged `internal/battencli` code uses the parent's numbering contract as written, and the two sides' additions do not conflict.

## Rejected

- Numbering contract: `overviewEntryNumber` goes through `loomrecipe.Rows` and `loomrecipe.EntryNumbers` as the parent wrote them.
  A row absent from the result gives `ok == false`, and the stage then shows the row alone.
  A review segment's rows share one number, which `TestRenderOverview` pins with the `6.6 Plan-Burler-Review` case.
  `Rows` now accepts `shedrun.RecipeBatten`, which is what `7. Land` and `6. Run-Shed` rely on.
  `internal/loomcli/checklist.go` is the other consumer of `EntryNumbers`, so both panes draw the same number for the same row.
- Per-poll cost: `Rows` parses the embedded recipe on each call, so a frame parses it a few times per run.
  That is CPU only.
  The frame stays lockless and write-free, since `resolveChildRecipe` only reads the child seed and `childPair` reuses the parent-side resolution cache, so the `frame` doc holds.
  The new `role_prefix` and `on_reroute` recipe keys do not change what `Rows` returns, which is name, segment, `OnStuck` and engine.
- Cached child seed: an absent seed is re-read each poll and an unresolved pair is not cached, both read-only.
  `TestOverviewSource_ReadsThePrimesRunsFromDisk` covers the cache after the seed is removed, a seedless pair and an unresolved pair.
- Widths and back-off: the stage cell is still padded to the widest cell, and the orch line is drawn outside the columns.
  Every time cell still goes through `FormatMinutes`, so the back-off claim in `cli.go` holds.
  The parent's `FormatSeconds` is for closed spans and `internal/battencli` does not use it.
  The `cli.go` doc comment describes both sides' lines.
- Test economy: no merged test duplicates another.
  `TestRenderOverview` covers the render of orch line, numbers and runs, and the source test covers recipe resolution.
  The orch line above a numbered run is two independent lines, and `TestRenderOverview` covers the orch line above a run.
- Budget: the `internal/battencli` row of `cmd/lyx/testdata/test-budget.yaml` is 83, the count of top-level tests there excluding the two `TestMain` functions.
  The parent's `internal/loomrecipe` row is raised to 56.
