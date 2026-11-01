```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D3 (re-run is idempotent vs which runs are held): held. On a re-run the subject is already owned here, so no new hold is written, and the holds already standing are not lifted.
- D1 × D5 (re-run vs the run report): broken. See finding 1.
- D2 × D5 (the reconcile vs `landed`): held. `localRuns` is read before the reconcile and `movedRuns` after it.
- D3 × D4 (hold vs release): held for task runs. Ambiguous for a nested beam's pawl run. See finding 2.
- D3 × D4 (owner write fails vs `beam go` refusing a beam owned elsewhere): held. The holds stand and `beam go` names `lym adopt`. Re-running adopt finds the entries already there.
- D3 × D5 (a run that is both held and owned elsewhere): broken. See finding 3.
- D3 × dobby's owner-pause recheck: broken in part. See finding 4.
- D4 × dobby's lift trigger (two starts after a release): held. The driver lock refuses the second driver. See finding 5.
- D3 × D6 (holds vs released tasks starting): held. The two act on disjoint sets.
- D3 × D1 step order (hold written before the owner write, `LivePair` read before it): held.
- Constraints `PATTERN-dobby-orch-pair`, `PATTERN-dobby-starts-runs` and `PATTERN-tolerant-records` against D3 and D4: held.

**Findings**

1. **MEDIUM — D5 × D1, acceptance criterion "a second `lym adopt` ... reports the same run states".**
   - `landed` means a run that was not done in this machine's records before the reconcile and is done after it.
   - On the second adopt that record is already done before the reconcile, so the run is not reported as landed. It is not reported at all, because only runs not done are listed.
   - So the second report differs from the first, and the criterion cannot hold as written.
   - Fix: weaken the criterion to "the same states for runs not done", or define `landed` so it lasts beyond one call.

2. **MEDIUM — D4 × D3, a nested beam's pawl run under root.**
   - A nested beam's pawl run has the prime as its parent worktree.
   - `movedRuns` for root lists "a beam below an adopted beam or root".
   - So the pawl run falls under both `lym beam go root` ("a child run of the worktree") and `lym beam go &lt;beam&gt;` ("its own pawl run").
   - The criterion "`go root` leaves the hold of a nested beam's runs standing" does not say which of the two releases the pawl run.
   - Fix: pick one owner for the pawl run's hold, presumably `go &lt;beam&gt;`, and say that `go root` excludes it.

3. **LOW — D3 × D5, a run that is both held and owned elsewhere.**
   - D3 excludes only a pawl run whose beam's parent is not owned here. Take a nested beam owned elsewhere, moved by an adopted root as a run: its subject (root) was owned elsewhere before the call, so D3's rule would hold it.
   - It then matches both `held` and `owned_elsewhere`, and D5 gives no precedence between them.
   - Fix: D3 should hold no run whose entry reads owned elsewhere after the owner write, or D5 should state the precedence.

4. **MEDIUM — D3, turning the owner entry into the hold, against dobby's owner-pause recheck (`internal/dobbyengine/owned.go` `recheckOwnerPause`).**
   - Once adopt's owner write lands, dobby's probe reads the owner as this machine and calls `LiftOwnerPause`.
   - `LiftOwnerPause` finds no owner entry, because it is now the hold, and returns nil.
   - dobby then sets `liftPending`, so a full pass is owed at once, even though the hold still covers the run.
   - The batten pass should then be refused as covered (exit code 78), so no work runs. But D3's claim that a covered run gets no full pass is false on this path, and the record does not mention it.
   - Fix: D3 states that dobby's recheck must not owe a pass when no owner entry was lifted, or that a refused pass is the accepted bound.

5. **LOW — D4 × dobby's lift trigger.**
   - `beam go` lifts the hold and also calls `StartFirstPass`. dobby owes a full pass at the same lift.
   - The run's driver lock makes this safe, but one of the two passes is refused each time.
   - D4's rationale ("what adopt's resume did") does not say whether the explicit start is needed when dobby is live.
   - Fix: state the double start as the bound, or rely on the lift alone when dobby is live.

**Acceptance criteria and constraints**
- D3 "dobby's owner-pause lift leaves a `HoldCaller` entry standing": holds as an entry check, but finding 4 shows a pass is still spawned.
- Every other criterion matched its decision, apart from the D1 re-run criterion (finding 1) and the D4 nested-beam criterion (finding 2).

**Tool calls**
- grep of `internal/dobbyengine/owned.go` and `passes.go` for the lift handling: my context did not have dobby's owner-recheck code.
- `sed` of `internal/dobbyengine/owned.go` lines 60–125: to read `recheckOwnerPause` in full and check how it interacts with turning the owner entry into the hold.
