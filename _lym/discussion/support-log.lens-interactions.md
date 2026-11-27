```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions that act on the same state, plus the criteria and constraints against the decisions they touch.

**Findings**

1. **BLOCKING — D10 × D6/D9. A held run is chased and then posted as severe.**
   - Under `unattended`, D6 and D9 have the orch pause a run instead of accepting a round with an open BLOCKING finding or cancelling the run. That hold is meant to be quiet and not severe.
   - D10 ends a chase only on "any transition of the run, a resume, a goto or an accept". A pause is not on that list.
   - So the `settle: orch` halt is still unsettled one interval after the chase. D10 then posts it as severe, class "no settling action open", through D7.
   - That contradicts D6's quiet `[operator] held` line and D9's rule that a cancel is not severe.
   - Fix: count a pause covering the run as settling the halt in D10, and add that case to D10's criterion.

2. **MEDIUM — D5 × D4 (and `push-retry-not-halt` D6). A resume during a deliberate wait blocks, then refuses.**
   - D4's pause wait holds the driver lock, and so does the peer's network wait.
   - D5 makes `lym loom resume` wait while the driver lock is held, then refuse with `driver_not_parked` once the keep-warm cap passes.
   - An orch resuming a paused or network-waiting run blocks for the whole cap and then gets a misleading refusal. D10's chase names `lym loom resume` as the way forward, which leads into the same refusal.
   - Fix: D5 should tell a driver that is deliberately waiting from one that has not parked, and refuse at once with a way forward that names the wait.

3. **MEDIUM — D2 × D3 × D4. The orch column of the board-claim row can never be reached.**
   - The table gives lym two settles, then "orch from the third stop", then severe at the "third halt at the row".
   - With D3 counting lym's two settles, the third stop is both an orch verdict and a severe verdict, so the orch column is dead.
   - The other rows go lym once, orch at the second, severe at the third, and stay consistent.
   - Fix: lower lym's bound to one, or make severe the fourth stop for this row.

4. **MEDIUM — D4 pause wait × `push-retry-not-halt` D3. Paused runs show up in the push summary.**
   - The constraint says D4's pause wait reuses the peer's wait record.
   - The peer's D3 posts a summary once "some run has kept a wait record" past `fabric.push_notice_after_min`. It lists every waiting run and holds back the all-clear while any run waits.
   - So every pause longer than 30 minutes would trigger a network summary and block the all-clear.
   - Fix: give the pause wait its own class that the summary skips, or a wait record of its own.

5. **MEDIUM — D1 × D12, `hubgeom.Autonomy` degrading to false.**
   - D1 refuses an unknown level with a way forward.
   - The exploration showed `hubgeom.Autonomy` "reads false on a failed read".
   - A typo such as `unatended` therefore silently runs with no autonomy, and D12's warning never fires because the task never resolves to `unattended`.
   - Fix: say where D1's refusal surfaces for a run or an orch.

6. **LOW — D3 × D2, escalation rows.**
   - D3 counts halt entries ("a Stuck with no `routed_to`") and the arbiter's resumes. Accepts count nothing.
   - A `hard-cap` escalation is settled by accept and is an awaiting stop, not a Stuck. Whether it ever reaches the table's "third halt at the row" is undefined.
   - Fix: say whether an escalation's awaiting stop counts as a stop at the row.

7. **LOW — D10 × D7 × D8, posting record and reminders.**
   - D7 keeps "posted once" in the driver's settle record. D10's severe post comes from the dobby, and the record does not say where its "once" is kept.
   - It is also unclear whether D8's reminders cover a halt that D10 escalated, since its verdict stays `orch`.
   - Fix: name where D10's post is recorded, and that D8's reminders cover it.

**Pairs checked**

| Pair | Result |
|---|---|
| D3 × D4, retry counting against the lym bounds | broken (finding 3) |
| D3 × D4, pause hold excluded from the count | held |
| D3 × D6, arbiter continue counted, accept not counted | held |
| D3 × D2, hard-cap and escalation counting | unclear (finding 6) |
| D4 pause wait × D6/D9 orch pause | held |
| D10 × D6/D9, held runs chased | broken (finding 1) |
| D5 × D4 pause wait and peer network wait | broken (finding 2) |
| D4 pause wait × peer D3 summary | broken (finding 4) |
| D7 × D8, prefix mention without a duplicate relay | held |
| D7 × D10, posting record | gap (finding 7) |
| D8 × D10, reminders after escalation | gap (finding 7) |
| D8 × D9, `[operator] held` stays quiet and reaches Slack | held |
| D1 × D7/D10/D12, level read | held |
| D1 refusal × `hubgeom.Autonomy` degrading to false | broken (finding 5) |
| D2 network row × peer D6, network waits uncounted | held |
| D6 × D2 table, budget row | held |
| D9 × D2, unattended severe and question columns | held |
| D11 × D10, a wait by design is never chased | held |
| D12 × D7, relay drop warned of | held |

**Acceptance criteria against decisions:** the D4, D3, D7 and D8 criteria match their decisions. The D10 criterion lacks the pause case from finding 1. The D5 criterion lacks the deliberate-wait case from finding 2.

**Tool calls made:** none.
