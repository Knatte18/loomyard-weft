```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop or recovery step the cards add, I checked the next step it names against the state it leaves.

**Findings**

1. **MEDIUM — Card 10, D6 replay rule.** In `stalemated`, the "`StalemateRounds` is 0 → false" test comes before the "own escalation record holds `stalemate` → true" test.
   - Sequence: a round escalates with cause `stalemate` and awaits. The operator sets `review_stalemate_rounds` to 0 instead of deciding, then resumes.
   - The replay then skips the stalemate branch. Below the soft-cap window, a CONTINUE verdict falls through to `ensureFocus` and a counted Stuck (`internal/shedadapters#Bouncer.settleUndecided`, bouncer.go:1361-1375). The next round runs without an `accept` or `continue`.
   - The pending escalation is left behind on a round that is no longer the latest, so `PendingEscalation` stops seeing it (circling.go:165-185).
   - D6 says a round whose pending escalation holds `stalemate` "escalates with that cause again". Fix: put the own-record check first, or state why the order is safe. No row covers it either: Card 10's replay row runs at count 2.

2. **LOW — Card 8, the verb's builder-error exit.** When `BuildRoundContext` errors, the verb prints the error and exits 1 with no way forward. Every other refusal the card adds ends in "way forward: rewrite the review file ...".
   - That error is a non-absence read failure of the snapshot record, fixer-report, ask record or decision record.
   - A reviewer self-checking gets exit 1 with nothing to act on, while its own gate entry (Card 7 path) logs the same error at Warn and runs the round full.
   - So the verb refuses where the gate would not. Fix: on a builder error, fall back to the shape checks alone, matching the producer's Warn-and-full path, or name a way forward.

3. **LOW — Card 8 against Card 7, verb and gate drift.**
   - The gate's context is built once, in `roundProfile` at round start.
   - The verb rebuilds it at check time from the live decision record (`addedDecisionDigests`). An entry added through `lym loom decision add` mid-review is listed by the verb but not by the gate.
   - A reviewer that quotes that title in `resolved` gets "ok" from the verb and a round-check refusal from its gate. The gate's next step ("rewrite the review file") still progresses, and the cap lets it through, but the self-check has misled it.
   - Fix: state the accepted bound in Card 8, or have the verb read the titles as of round start.

4. **LOW — Card 12, the `LoadConfig` refusal.** Its next step, "set it to 0 … or to a positive integer in loom.yaml", ignores the layer.
   - `CategoryNone` leaves the key open to the local layer, which a task's `run_settings` writes.
   - A negative value set there is not reached by editing loom.yaml.
   - `reviewbudget.Route` (internal/reviewbudget/reviewbudget.go:103-119) already words the layer-aware way forward.
   - The same flaw exists in the `review_circling_checkpoint` refusal at internal/loomengine/config.go:1389-1390, so this copies existing practice.

**Tool call**
- Read internal/loomengine/config.go:1375-1400 and `repairReportBeforeGate` (internal/burlerengine/engine.go:106-118). I needed the existing config-refusal wording for Card 12's way forward and the fix-gate wrapper's re-prompt text for Card 2's refusal path; my context had neither.
