```yaml
lens: fix-affected-set
outcome: reported
```

Checked whether the fix covers every member of each set-based finding (F1–F5). Unchecked: none.

1. **LOW** — Card 9 `batchGateCalls` (09-batch-attempt-strand.md:34-36) against card 8.
   F3 now gives the gate calls one renderer, but there is a third rendering.
   Card 8's batch prompt renders the same card and batch gate calls on its own, through `runplan.RenderCardGates` and `runplan.RenderBatchGate` with the main packages on the last batch (08-weaver-stencils.md:37, 42).
   Those helpers call `gate.AgentGateCall` and `AgentGateCallWithMainPackages` (internal/runplan/cardgate.go:28, 61-63), while card 9's calls use `GateCall("card", <card id>)`.
   So the gates the prompt tells the agent to run and the gates the strand enforces can still drift, which is the drift F3 set out to remove.
   Card 8 was not brought under `batchGateCalls`, and no sentence ties the two together.

2. **LOW** — Card 12's new stop-record bullet (12-run-loop.md:59-61).
   F5's premise is that every Stuck return must state whether it records a stop.
   The new text covers batch, needs-parent and verify stops, plus a failed stop after a last-rung failure.
   It names plan-conflict and config or state faults as setting none.
   It leaves out other Stuck routes in the same card:
   - a fixer whose stop failed (line 55, row line 84);
   - a runner crash in the verify, which "stops the row" (line 56);
   - a failed per-batch commit (row line 75);
   - a failed final record commit (line 58).
   None of these says whether `st.Stop` is set, so whether `reenter` runs after them is still open.
   The verify runner crash is the clearest case, because it is a row stop in the verify stage.

3. **LOW** — Card 23 `RunInFlight` decoding through `outcomeRecord` (23-merge-in-warning.md:26-27), together with `seedWeaverState`.
   F4's premise is the set of places that restate `outcome.yaml`'s keys.
   `seedWeaverState` belongs to `internal/fabriccli_test` (card 23 Create) and writes a Weaver `outcome.yaml` with a given outcome.
   It cannot use the unexported `weaverengine.outcomeRecord` or `writeOutcome`, so it has to spell out the keys again.
   That leaves one restatement outside the single shape the fix introduced.
   Card 23 does not say how the seed stays in step with `outcomeRecord`.

I also checked whether F2's entry removal of tracked `outcome.yaml` and `summary.md` reaches `checkCleanTree` (card 10) or the reset's foreign-path refusal (card 14, step 6).
Every attempt already saves `_lym/weaver/state.json` before `checkCleanTree` runs, so a run-directory change cannot be what that check flags.
I raised no finding there.

Tool calls beyond my context:
- grep of every card for the lost / same-rung allowance: reran F1's search across all cards, which my context held only for cards 5, 12 and 13.
- grep of every card for gate-call rendering and `outcome.yaml` keys: reran the F3 and F4 searches; card 8 was not in my context.
- grep of cards 10-16 and the overview for Stuck and `StopRecord` sites: reran F5's search across the Stuck routes.
- grep of cards 10 and 14 for `checkCleanTree`, the run's own paths and archive handling: checked F2's removal against readers my context lacked.
- grep of `DirtyPaths` in websterengine, fabricengine and verifytree: checked whether the clean-tree check sees the run directory.
