# Card 5 — review-gate-rounds

**Edit:**
- `internal/parentreview/gate.go`
- `internal/parentreview/gate_test.go`

**Uses:**
- `plan:internal/parentreview#Store.RejectedRounds`
- `internal/parentreview#Store.BeginRound`
- `internal/parentreview#Store.MarkConsumed`

**Intent:**
Make the parent review every rewrite after a reject, up to a reject cap, and escalate at the cap instead of letting the rewrite through (Decision 5).

`GateConfig` gains `Cap int`, the number of rejected rounds that fails the entry, passed into `OpenSpec.Cap` when a request opens.

The per-arrival closure reads the latest round before opening anything:

- no round or no request: open a request, as today;
- an approve verdict, superseding or not: pass;
- a reject verdict, when `RejectedRounds` is at least `Cap`: fail at once with `GateResult{Terminal: true}`, opening no request and consuming nothing;
- an unconsumed reject below the cap: consume it and fail with today's findings naming the review file, so the engine re-prompts the live writer;
- a consumed reject below the cap: `BeginRound`, then open the next round's request with a fresh brief and delivery prompt;
- an expired request: pass; an open request with no verdict: today's wait-bound, notify and prompt logic.

The terminal findings are one line naming the slug, the rejected-round count, the latest review file, and the way out: `lyx loom review approve <slug>` from the parent, then `lyx loom start` in the task worktree.
That line becomes the `blocked` reason through card 3's `GateOutcome.Reason`.

The final closure passes only on no reviewer, no round or request, an approve, or an expired request.
An open request with no verdict is marked expired and passes with today's timeout Warn, instead of returning pending.
Any reject on the latest round, consumed or not, fails with `Terminal: true`: at the cap with the cap line above, below it with a line naming the review file and saying the re-spawned writer is re-prompted with it on `lyx loom start`.
A consumed reject fails too because the writer's rewrite after it was never reviewed; this is the bound's "passes only on" list, not a widening.
The final closure still opens nothing and consumes nothing, and its reject counts toward the cap through the store as the gate's does.

Update `gate.go`'s file comment, `GateConfig`'s doc and the two closures' docs.

Tests in `gate_test.go` over a real store in a temp dir: a reject followed by a passing arrival opens round 2 and returns pending carrying the delivery prompt; an approve on round 2 passes; with `Cap` 3, the third reject fails terminal with a reason naming the slug, `3`, the review path and `lyx loom review approve`; a fresh `NewGate` over the same store after two rejects still fails terminal on the third, and a fresh gate built between rejects (an attach) neither resets nor advances the count; a superseding approve on the latest round passes; a resume's first arrival with the cap's reject still latest fails at once and opens no round; the final closure fails on an unconsumed reject and on a consumed one, passes on an approve, and expires and passes an open unverdicted request; a timeout at the wait bound still passes with a Warn.

**ImpactSummary:** Rewrites the parent-review closures' round handling; only Discussion-Write's parent-review entry uses them, and its recipe wiring changes in card 7.
