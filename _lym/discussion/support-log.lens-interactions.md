```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that share state, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D2, shared spec builder. Held, if D1 passes "runs present" explicitly. If D1 instead feeds the builder the strand list it read before its `ReplaceStrand`, the builder sees no runs strand and returns the overview spec. Finding 3.
- D1 × D2, shared status sidecar. Held. D1 writes the new guid and build, and loom start's `resolveStatusStrandAction` then keeps the strand.
- D1 × the existing `ensureStatusStrand`, a concurrent loom start in the same session. Broken (finding 1).
- D1's failure path × the next `lym beam start`. Broken: the checklist is not retried (finding 2).
- D2 × D3, window name. Held when the seed is readable: D2 names the group by the recipe, and D3 prints that recipe. When the seed is unreadable, D2 falls back to the window `loom-status` and D3 renders no line. Consistent.
- D2 × D3, overview anchor case. Held. D3 renders only inside the runs strand, so the status strand is always in its beside-runs placement, and the name D3 prints matches.
- D1 × D3. Held. D3 reads only the seed and status files, not the strands.

**Findings**

1. MEDIUM — D1 with the existing `ensureStatusStrand`: two adds of the same role can race.
   - D1 runs from the pawl run's bring-up row, in `ensureRunsStrand`, and takes no loom bootstrap lock.
   - Suppose an operator runs `lym loom start` or resume at the same time. Between D1's `ReplaceStrand` and its re-add, the loom start can read the strands and find a runs strand and no `loom-status`. Its action is then add.
   - Reed refuses an add whose role another strand already holds. If D1 adds first, the loom start's `AddStrand` fails. That failure is fatal in `ensureStatusStrand`: its add branch does `return err`, unlike the replace branch, which only warns. So `lym loom start` fails.
   - The record does not mention this window. Either D1 should treat "role already held" as success, or the record should state that loom start's add tolerates a lost race.
   - Evidence: `sharedbootstrap.go`'s `ensureStatusStrand` add path, and the comment in `bootstrap.go` that the reed add "refuses an add whose explicit role another strand already holds".

2. LOW — D1's failure path is not retried, and the record does not say so.
   - D1 says a failed re-add is a warning and the runs strand stays. On the next `lym beam start`, `ensureRunsStrand` finds a current runs strand and no `loom-status`, and returns early. Nothing re-adds the checklist until the next `lym loom start`.
   - The Goal says "always shows". The record should state this bound, or the warning should name `lym loom start` as the way forward. As written, the warning names `lym beam start`, which does not fix it.
   - D1 also does not say whether the runs sidecar is still written when the re-add fails. If it is not, the next beam start replaces the runs strand again.

3. LOW — D1/D2: the builder's input in D1 is underspecified.
   - D2's builder chooses the anchor from "the runs strand in the session". D1 calls it right after `ReplaceStrand`, using the strands list it read before the replace, which has no runs strand.
   - The record says "the beside-runs spec of D2" but does not say D1 forces the beside-runs case. A plan that reuses the strand list it already read would get the overview spec. Reed would then refuse it as a second overview, and the add-failure warning would fire every time.
   - Fix: D2's builder takes the placement as an input, not the strand list, or D1 states that it asks for the beside-runs case.

**Criteria and constraints**
- The D1 acceptance criteria cover success, add failure and the unchanged cases. None covers a concurrent loom start (finding 1).
- The D2 criteria are consistent with D1 and D3.
- The D3 criteria are consistent with D2's fallback.
- The constraint "never fails `lym beam start`" holds with D1. The record does not say whether `lym loom start` can still fail under finding 1's race.

**Further tool calls:** none.
