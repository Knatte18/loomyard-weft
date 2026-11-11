```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: I traced each state, setting and file the record introduces or changes through the code that already writes or reads it.

**Paths traced, per decision**
- D1: the assistant keys are open to `run_settings` through the local layer (config.go:657-661, `none`). Survives.
- D2, generation start: `startNextGeneration` archives the run dir and `seedCall` seeds round 1 again (amendment.go:42-59, archive.go:67-76, bouncer.go:1621-1651). This breaks the record's claim that round numbers carry on.
- D2, goto back to Discussion-Write while the perch is unsettled: the clear does not fire, the generation stays the same and the next round is N+1 (bouncer.go:367-369, 238). This breaks the claim that a goto starts a new generation.
- D2, `lym loom start` or resume on a live run: "a goto or a resume restarts nothing" (bouncer.go:238). Breaks the claim that a fresh `lym loom start` starts a generation, except for a brand-new run.
- D2, what the generation is read from: only `generation-record.json` in the run dir, through the unexported `readGenerationRecord` (settlerecord.go:24, 176). The record does not name it.
- D3, the clear path (`entryClear`): it archives and re-seeds round 1, but `CarryRecords` keeps the generation record (bouncer.go:458-470, archive.go:74, settlerecord.go:261-282). The (generation, round 1) key gets reused. Breaks.
- D3, ask records under `.lym`: `archiveRunDir` moves only the tracked run dir, so these records outlive every archive. That is intended for D3's restart rule, and it is what makes the clear-path reuse above possible.
- D3, a retried review attempt with a new token in the same round: keyed by generation and round, never by token. Survives.
- D3, the `NotifyParent` run tag: the pair's teardown drops unread asks (notify.go:43). Survives.
- D3/D4, which orch gets the ask and which passes the guard: both are re-resolved by liveness at every call (notify.go:44, decision.go:685, 1695). Breaks when liveness changes between the ask and the answer.
- D4, the copy beside the review: `parseRoundFileName` reads `round-N-review.<x>.md` as a kind other than `review.md`, and `ResolveRound` stats only the exact report name (round.go:23-43, 128-142; settlerecord.go:211). `liveSettleRecord` and the round count still read correctly. Survives.
- D5, the chair's timeout: it is `review_timeout_min`, or shuttle's `run_timeout_min` when that key is `0`. The existing cap subtracts the start allowance and one await cap, and it skips the seat when no room is left (review.go:64-83). The record names no formula and no no-room outcome. Breaks (deferred to Plan).
- D5, the keep-warm cap from the chair's strand (wiring.go:1287). Survives.
- D6, `decision add` during Discussion-Review: it is refused only while Discussion-Write is running (decision.go:893-895). Discussion recipes' record engine is not `RecordWrite`, so `checkDarnRecord` is skipped (decision.go:1535-1541, recipekeys.go:171-183). Survives.

**Findings**

1. **BLOCKING, design.** `decision-record.md` D2, "Which rounds ask". The generation premises are false, so a rewritten record can skip the parent.
   - A new generation archives the run dir and seeds round 1 again, so round numbers restart (amendment.go:42-59, archive.go:67-76, bouncer.go:1621-1651).
   - A goto back to Discussion-Write over an unsettled, escalated round stays in the same generation and sends the rewritten record to round N+1 (bouncer.go:238, 367-369). That round asks only if the parent's previous answer held findings, which breaks D2's own aim: "every rewritten or amended record goes to the parent".
   - A resume or `lym loom start` on a live run starts no generation (bouncer.go:238).
   - The record never says the generation number is read from the run dir's `generation-record.json` (settlerecord.go:24, 117-125, 176). It leaves that source for Plan to find.

2. **MEDIUM, design.** D3, ask record keyed by generation and round. The Bouncer's clear path (`entryClear`, bouncer.go:458-470) archives and re-seeds round 1 without bumping the generation, because `archiveRunDir` carries the generation record into the fresh dir (archive.go:74, settlerecord.go:259-282). The bouncer.go:465 comment names a commit-seam failure followed by a resume as one way onto this path.
   - The re-seeded round 1 finds the old (generation, round 1) ask record, which is already `done`, and sends no ask.
   - The old parent report is copied beside the new round-1 review, and D7 makes the chair keep its stale findings again.
   - The record does not mention this path.

3. **LOW, design.** D3/D4, parent resolution. The ask's receiver and the answer guard's parent are each re-resolved by liveness at every call (notify.go:44, decision.go:685, decision.go:1695, nearestlive.go:26-56). The receiver also takes reed's recorded name (notify.go:171-178), while the guard forms the name with `agentname.Format` (nearestlive.go:52).
   - If a beam orch comes up or goes down between the ask and the answer, the orch that got the ask is refused.
   - The refusal's way forward ("the run's parent orch answers") points back at that same orch, so the round waits out the deadline.
   - The record states no bound for this.

4. **MEDIUM, design.** D5, the deadline cap. The existing cap the record copies (`CapAssistTimeout`, review.go:64-83) reads the chair's timeout as `review_timeout_min`, or shuttle's `run_timeout_min` when that key is `0`. It subtracts the start allowance and one await cap, and when no room is left the seat does not run, with a reason.
   - D5 names no formula. The deadline's clock starts at the ask, before the table starts, while the chair's clock starts at its launch.
   - D5 leaves no stated room for consolidating after the await returns.
   - D5 does not say what happens when the cap leaves no time, or when `review_timeout_min` is `0`.
   - All of this is left for Plan to find out.

**Tool calls**
- decision.go:1528-1545 and the `RecordWrite` greps: whether `checkDarnRecord` gates `decision add` on discussion recipes; my context did not hold `rowWritten`'s body.
- shedadapters/round.go, `parseRoundFileName` and `ResolveRound`: whether a copy beside the review would be misread as a round file; not in my context.
- `hubgeom.NearestLiveOrch` and `orchroute.FromPair`: whether the guard's parent name and the ask's receiver name are formed the same way; not in my context.
- loomrecipe `RecordEngine`: which record engine the discussion recipes use, for `rowWritten`; not in my context.
