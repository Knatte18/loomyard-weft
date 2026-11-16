```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked each pair of decisions that act on the same state, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D12 × D13 (a strand left live after a halt; decision notice held back): broken.
- D6 × D12 × D7 (strand of a timed-out or failed attempt; attach on re-entry; arbiter fix in the same tree): broken.
- D1 × D13 (which implementer counts as "active"): broken.
- D13 × D3/D12 (renumber bound against Weaver beginning a batch; the lock each one takes): broken.
- D8 × D12 (journal append, then recording a finished report after a crash): broken (LOW).
- D5 × D6 × D7 (same-rung machinery retry inside the re-entry attempt): ambiguous (LOW).
- D3 × D11 × D9 (a rename rewrites pending cards, then the plan directory is committed per batch): held.
- D7 × D3 (re-entry checks use the batch's card set and start commit recorded in state): held.
- D7 × D11 (an arbiter fix carries no report or renames; the drift check catches it at begin): held.
- D9 × D1 (spec Out line allows commits during the run): held.
- D9 × D6 (the per-batch `_lym` commit against the unclean-tree check): held. `_lym` is weft-side and excluded on the warp side.
- D9 × D10 (`summary.md` under `_lym/weaver/` is committed with the batch commits): held.
- D10 × D5 × D7 (fixer climbs the ladder; after a verify stop, one fixer on the last rung): held.
- D4 × D10 (row `gates` on batch and fixer strands): held.
- D2 × D6 × D13 (plan-conflict reroute; shared conflict path): held.
- Acceptance criteria × D1/D2/D5/D6/D7/D10: held. The criteria's lists of named paths are subsets of D2's, which is consistent.
- Constraint PATTERN-shuttle-provider-seam (stop only through shuttle's stop verb) × D6/D12: broken, as part of the second finding.

**Findings**

1. **MEDIUM · design · D12 × D13.** A halted run's live Weaver strand is held back from decision notices (D13), but on re-entry D12 attaches to that same live strand instead of starting a fresh one. The decision recorded during the halt never reaches the agent that resumes.
   - Webster's hold-back rests on "the fresh session the run's re-entry starts carries the decision" (`internal/loomcli/decision.go:437-448`).
   - D12 (`decision-record.md:131`) attaches to a live in-flight run, so that premise is false for Weaver.
   - Fix: on re-entry, either deliver held-back decisions to the strand D12 attaches to, or stop that strand and respawn.

2. **MEDIUM · design · D6/D7/D12.** No decision says Weaver stops a failed attempt's strand, for example one that timed out (D6) or whose gates still fail, before it climbs the ladder or returns Stuck.
   - After a batch stop, the arbiter commits a fix in the task tree while that strand may still be live.
   - On re-entry, D12 runs first and attaches through `FindRun` to "a live run of the in-flight attempt", so it waits on or records the stale agent instead of running D7's checks.
   - Two agents can also share the tree during the arbiter's fix.
   - PATTERN-shuttle-provider-seam requires stopping through shuttle's stop verb (`pattern/PATTERN-shuttle-provider-seam.md:51-55`), but no decision places that stop.
   - Fix: D6 stops the strand through shuttle's stop verb before the next attempt or the Stuck, and D12 attaches only to the attempt state records as in flight, never to one already failed.

3. **MEDIUM · design · D1 × D13.** D13's seam reads "the active implementer's" state without saying what decides "active". D1's guard runs only when the implementer row is stepped.
   - If "active" is read from `tapestry.implementer`: Weaver builds cards, the run reroutes to Plan-Burler-Review (D6), then the setting is flipped to `webster`.
   - The plan gate (`internal/loomshed/gates.go:254-276`) and the renumber bound would then read Webster's empty state, freezing no done cards. That is the outcome D13's rationale exists to prevent.
   - D1's guard fires only later, at the implementer row, after the plan has already been rewritten.
   - The record also does not say what a corrupt or missing state file of the other implementer lets through, for either D1 or D13.
   - Fix: D13's seam decides "active" from which implementer has begun batches in the current generation, not from the setting, and an unreadable state refuses.

4. **MEDIUM · design · D13 × D3/D12.** The renumber verbs take Webster's state-mutation lock before reading the bound, so they cannot interleave with `begin-batch` (`internal/loomcli/renumber.go:58-63`, `websterengine.AcquireStateMutation`).
   - D13 moves only the bound read to Weaver's state.
   - D12 names a run lock held for the producer's whole call, but no decision has renumber take that lock or a shared state-mutation lock.
   - A renumber can therefore pass its bound check while Weaver records a new batch's card set (D3), moving cards that batch has begun.
   - Fix: D13 says the renumber verbs take the lock Weaver's batch-begin writes under.

5. **LOW · design · D8 × D12.** Go appends the journal entry when it records a report (D8), and after a crash D12 "records a finished report that was not yet recorded".
   - If the crash falls between the journal append and the state write, recording the report again appends a second journal entry for the same attempt.
   - Fix: state the order, or make the append keyed by batch and attempt so it does not duplicate.

6. **LOW · consistency · D5/D6/D7.** D6 gives a machinery loss "one fresh attempt on the same rung", but does not say whether that is once per rung or once per batch, or whether it applies to D7's single re-entry attempt and to D10's fixer strands.
   - D5's bound says "plus the same-rung retries D6 allows", in the plural.
   - Fix: D6 states the retry is once per attempt, and that it applies to re-entry attempts and fixer strands.

I made no tool calls.
