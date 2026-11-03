```yaml
lens: interactions
outcome: reported
```

**Interactions lens report: the decision record and support log of shed-sub-run-block**

Pairs checked:
- D1×D2, block row keys against expansion: held. Expansion drops nothing that D1 allows.
- D2×D9, empty `on_done` rewritten against cartoon's empty Discussion-Bouncer Done: held. The Burler rows' `on_done` values are not empty.
- D2×D4, the exit rewrite against when the block counts as Done: held.
- D2×D3, the block row's `on_done` becoming the sub entry while the original is kept as the skip target: held, as long as D3's skip target is read from the pre-expansion `on_done`.
- D3×`enter` skip chain, a skipped block followed by Drift-Baseline's own `drift.enabled` gate: held. Chained skips still work and the chain cap is the length of the producer list.
- D3×pause before/after on the block row and the next row: held.
- D3×D5, `always` building a nil gate against `EnteredRow` and `markEntered`: held.
- D3×D4, goto onto the block row against a halt inside the block: held.
- D4×resume, re-calling the halted inlined row: held.
- D6×`appendOrFold` folding: held. One producer always has one block path.
- D6×goto entries: broken (F1).
- D6×D2, nested blocks: broken in wording (F4).
- D7×D3, progress after a pass-over: held. Drift-Baseline is on the main line.
- D7×D2, `EntryNumbers` by file order with inner rows placed right after the block row: held.
- D7×D2, the segment-collision refusal against keeping segment budgets: held.
- D8×D2, a sub-recipe setting `profile_recipe`: gap (F3).
- D8×D10: held.
- D9×D10, the two equality pins: broken (F2).
- D9×D11, cartoon in loomrecipe's table but not seedable: held. `IsLoom(cartoon)` is false, so `New` refuses it.
- D10×loom contract on the expanded rows, where the block row has no engine: held.
- D11×D2, arras including itself: held. The self-inclusion refusal covers it.
- Constraint "tapestry runs unchanged"×D3 and D6: held. An unset skip target and an omitted field leave tapestry's behaviour as it is.

Findings:

- **F1 — MEDIUM — D6 × D11 (goto stamping).** D6 says a goto entry for an inlined row carries the block path. Goto writes its entry from a Routing projection, and neither D11's new projection nor `RoutingOf` is said to carry the block path.
  - Evidence: `internal/shedengine/goto.go` appends `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto, …}` at lines 136 and 250.
  - The `GotoRequest` doc says Producers needs only Name, Segment, OnDone and OnStuck, "so a Routing.Producers projection works".
  - D11 adds "a routing projection from an already-expanded recipe" but does not say it copies the block path.
  - Fix: say the routing projection carries `ProducerDef`'s block path, and that Goto stamps it from there.

- **F2 — MEDIUM — D9 / D10 acceptance (the two pins disagree on edges).** D10 pins arras's expanded rows "equal to tapestry's rows plus the Cartoon row, edge for edge". That cannot hold.
  - Preflight's `on_done` is Cartoon in arras and Discussion-Write in tapestry.
  - D9 also says cartoon has "the same edges" yet keeps the Bouncer's Done empty.
  - Fix: state the exceptions in both pins:
    - arras Preflight → Cartoon;
    - Cartoon's `on_done` is Discussion-Write after expansion;
    - cartoon's Discussion-Bouncer Done is empty before expansion.

- **F3 — LOW — D8 × D2 (a sub-recipe's `profile_recipe` is ignored).** D2 refuses a sub-recipe that sets `contract`, `verify_source` or `amendment_target`, but D8 adds a fourth run-level key, `profile_recipe`. A block recipe that sets it would be ignored without notice.
  - Fix: add `profile_recipe` to D2's refused keys.

- **F4 — LOW — D6 × D2 (nested blocks).** D6 says "the block row's own entries carry none". In a nested block, the inner block row is itself an inlined row of the outer block, so by D2 it carries the outer path.
  - Fix: say a block row carries the path of the blocks that enclose it, not its own name.

- **F5 — LOW — D2 (inlined `on_done` leaving its recipe).** D2 refuses an inlined `on_stuck` or `on_reroute` that names a row outside the sub-recipe, but not a non-empty inlined `on_done` that does so. Such a row would jump out of the block past its exit, and the build would still accept it as valid.
  - Fix: refuse a non-empty inlined `on_done` that names a row outside the sub-recipe too.

Tool calls:
- Grepped `internal/shedengine/goto.go` and the `GotoRequest` callers to see how goto writes its history entry and which producer projection it is given. My context showed only the first lines of goto.go's header.
