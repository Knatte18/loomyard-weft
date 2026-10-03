# Card 8 — bouncer-circling-resolution

**Edit:**
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/doc.go`

**Create:**
- `internal/shedadapters/bouncer_circling_test.go`

**Uses:**
- `internal/shedadapters/circling.go`

**Intent:** Make the Bouncer act on the operator's circling decision and tell the operator how to give one (D7).

`BouncerConfig` gains two optional told values:
`Slug string`, the task slug the Awaiting Reason addresses the verbs with, empty meaning the verbs run without a slug argument;
and `Bounces func() (count, budget int, ok bool, err error)`, reporting the segment's spent bounce count and budget, nil meaning unknown.

`settle` on `verdictCircling` reads `readCirclingDecision` for the round:

- No decision: Awaiting with the ledger pointer, as in card 4, and the Reason names the round, `lyx loom circling accept <slug>` and `lyx loom circling continue <slug>`, then `lyx loom start` to resume.
  When `Bounces` reports `count >= budget`, the Reason adds that `continue` will block on the bounce budget and that `lyx loom goto --to <Name>` from that block gives the segment a fresh budget.
  A `Bounces` error logs a Warn and omits the budget sentence.
- `continue`: Stuck to the round producer with the ledger pointer, after `ensureFocus(round+1)`, exactly as `CONTINUE`; it spends budget like any Stuck.
- Pending accept: `settleCirclingAccept`, then Approve, Commit and Done with the ledger pointer, exactly as `CONVERGED`.
  A failed settle write is returned as an error before Approve runs, so an accept never passes without its settled record.
- Settled accept reached in `settle`: unreachable, since the Call-entry clear intercepts it; it degrades with a Reason rather than settling twice.
- A malformed decision file degrades to Stuck with the read error as the Reason.

The Call-entry clear also fires when round N's verdict is `CIRCLING` and its decision is a settled accept, so a re-entry, including a resume after a failed Commit seam, archives the run directory and re-seeds instead of settling again.
The entry-time live-judge probe still runs before both.

The bound, stated in `Call`'s doc comment: an accept lets exactly one circling round pass without a converged judgment, skips no seam, and its record stays in the committed run directory; Done remains reachable only from a judged round whose review exists.

`doc.go`'s Bouncer bullet adds the decision handling and the Awaiting hand-off.

Tests in `bouncer_circling_test.go`, with fake Approve and Commit seams that count calls:
a `CIRCLING` round with no decision returns Awaiting, a Reason naming both verbs with the slug, and no shuttle Run;
with `Bounces` reporting the budget spent, the Reason names the budget block and `lyx loom goto --to <Name>`, and with budget left it does not;
a `Bounces` error still returns Awaiting without the budget sentence;
after a recorded continue, the Call returns Stuck with the ledger pointer and no seam runs;
after a recorded accept, the Call marks it settled, runs Approve then Commit once each and returns Done with the ledger pointer, and the next Call clears the run directory and re-seeds;
an accept whose Commit seam fails returns the error with the accept already settled, and the resumed Call clears and re-seeds without calling Approve again;
a malformed decision file returns Stuck and runs no seam.

**ImpactSummary:** Adds two optional `BouncerConfig` fields that every existing caller leaves zero, and adds the decision branch to `settle` and the Call-entry clear; CONVERGED and CONTINUE handling is unchanged.

**Commit:** 8: bouncer: resolve a circling halt from the operator's decision
