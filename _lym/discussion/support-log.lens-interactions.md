```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that act on the same state, and checked the criteria and constraints against them.

**Findings**

1. **BLOCKING — D1×D2: a failed or bypassed commit can land the replaced record without the trailer.**
   - D1 says replace "commits them in one commit carrying the replace marker of D2". It does not say what happens when that commit fails.
   - `add`'s commit-failure path keeps the written file. Its way forward is `lym fabric commit` (`decisionWayFabricCommit`, `wayCommit` in `decisionVerb`). That commit would carry no `Lym-Record-Replaced:` trailer.
   - `rollBackAppend` and `afterFailedCommit` also show that a status commit can land files the verb wrote. The replaced bytes could therefore reach HEAD in a commit with no trailer.
   - In both cases D2's bounded history still reaches the pre-replace versions. `CheckAddedDecisionsKept` then reports every folded entry as `added-decision-dropped`.
   - The run is then stuck. `checkDarnRecord` refuses `add`, and the gates fail. D1's own re-run is refused too, because working equals committed but the base hash has moved.
   - The D2 acceptance criterion holds only on the clean path.
   - Fix: D1 must restore the prior record on any commit failure, so no trailer-less version holds the replaced bytes, and give a way forward that re-runs replace, never `lym fabric commit`. The alternative is for D2 to key the reset on something a status commit cannot carry past.

2. **MEDIUM — D1×D2, `--amends-plan` roll-back.**
   - D1 reuses "`add`'s preconditions and roll-back".
   - `afterFailedCommit` decides "landed" through `committedLacksEntry`, which looks for `### Added after Discussion` headings. Replace adds none: with `added` empty, `committedLacksEntry` would index `added[0]` when HEAD holds no record. Even where it doesn't panic, it says nothing about whether the replaced bytes landed.
   - The roll-back cannot be reused as is. It needs a byte comparison of HEAD's record with the file.

3. **MEDIUM — D4×D5: one write stamps `revised` and `record_stored`, and a false warning can follow.**
   - D4 lets one upsert change the body and the record together. D5 warns when `revised` is later than `record_stored`, and its acceptance criterion says equal times give no warning.
   - The record does not say that both stamps take one clock reading in that write. Today `stampDates` stamps `revised`, and D5 adds a separate stamp on the record path.
   - Two clock reads give `revised` > `record_stored` by nanoseconds, so the warning fires on exactly the write D4 exists for.
   - Fix: D5 states that a write changing both stamps them with the same time. Alternatively, it compares at a coarser grain, or `record_stored` ≥ `revised` counts as current.

4. **LOW — D1 admission×D6 guidance, motif runs.**
   - A motif run ends at its Discussion approval, so D1 is never admitted while a motif run is live. Its added entries can be folded only through D4 on the Sketch.
   - D6's parent directive says only "change … after approval with `replace`", which reads as if a motif run's parent can fold in the run.
   - The Auto-mode assumptions mention `--record-file`, but D6's directive text should name D4 for a Sketch.

**Pairs checked**

| Pair | Result | Note |
|---|---|---|
| D1×D2 clean replace path | held | |
| D1×D2 commit-failure and status-commit paths | broken | finding 1 |
| D1×D2 `--amends-plan` roll-back | broken | finding 2 |
| D1×D3 | held | replace refreshes the board copy, and `StoreDecisionRecord` bypasses the run lock |
| D1×D4 | held | D4's held-entry refusal routes to D1, and D1 then reaches the board through D3 |
| D1×D5 | held | a replace on a raddle run stamps `record_stored` through D3, and promote has already happened |
| D1×D6 | held, with a wording gap | finding 4 |
| D2×D3 | held | the board copy has no history |
| D2×D4 | held | a board replace seeds a fresh run history, and held entries are refused |
| D2 × the existing `checkDarnRecord` and `committedLacksEntry` | held | both read through the bounded seam |
| D3×D4 | held | D3 writes only through the lock bypass, and D4 refuses held entries, so they never write the same entry at once |
| D3×D5 | held | |
| D4×D5 | broken | finding 3 |
| D1 base hash × D3 stale board copy | held | a mismatch refuses, which is the safe direction |
| D1 carry-over rule × D2 | held | |
| D1 × IdentityText and Bouncer re-entry | held | stated as a bound |

**Acceptance criteria and constraints**

- The D2 criteria depend on finding 1 and hold only on the clean path.
- The D5 equal-times criterion depends on finding 3.
- The other D1, D3 and D4 criteria are consistent with their decisions.
- The constraints `PATTERN-run-records`, `PATTERN-fabric-boundary` and `PATTERN-sole-parsers` are consistent with the decisions.

I made no further tool calls; my context held everything this lens needed.
