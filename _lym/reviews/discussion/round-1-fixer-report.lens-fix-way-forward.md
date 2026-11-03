```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward this fix added (refusals, next steps, rows, tests) against the state each one names.

no findings

The new D2 refusals are an empty `on_done` on a block row outside every block, and a block row whose `on_done` names itself or a row it inlines. Both fall under D2's rule that each refusal names the block and the way forward, and an author can clear either by setting or changing that row's `on_done`. The re-launch onto the block row is reachable: the block row comes before its inlined rows and its exit, so `gotoAdmitted` (`internal/shedengine/goto.go:338-341`) admits it from any later row. Clearing the entered row then makes `enter` evaluate the gate (`entry.go:30`). An inner block row with an empty `on_done` skips to the outer block's exit. That exit can never be empty, because the outermost block row must set `on_done`, so a pass-over always moves the run forward.

Outside my lens, for fix-claims: D4 says a reader finds the block's end as "the first entry whose block path does not include the block". A goto entry onto a row before the block, such as Preflight, also matches that rule, so the reader could take it for the block's end.

No further tool calls.
