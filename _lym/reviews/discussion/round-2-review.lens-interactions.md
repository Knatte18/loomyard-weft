```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I ran each pair of decisions that act on the same state against each other, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D4 (outcome entries under a Current or Affected verdict): broken. See finding 3.
- D1 × D5 (a Miss leads to a failed build, then a pass at the same HEAD): held. D5's exclusion and D8's queued-work rule stop the re-queue.
- D1 × D5 (Affected writes the map at H first, then the build fails): held. The map at H has gaps, which the gate runs, and each new landing costs one failed build.
- D1 × D6 (re-stamp and the environment stamp): held. A differing stamp is a Miss, so a re-stamp only happens when the stamp is equal.
- D1 × D9 (a map skipped because its lock is held, on a hub with fewer than two slots): held. The skipped map counts as queued work, and D8 starts no pass on queued work alone there.
- D1 × D10 (a discarded round leaves gaps): held. The next plan queues the gaps, and D8 leaves a dirty-skipped package out of queued work, so a dirty prime causes no restart loop.
- D4 × D9 (a run killed by a yield records nothing and is queued again): held.
- D4 × D9 (hand calls and spawn-marked tests): held. Spawn-marked tests are never measured, hand calls included.
- D7 audit × D8 restart (the audit ends a pass at the same HEAD with no queued work): held. No restart follows.
- D7 × D9 (the audit needs a slot by D9's rule; none on a hub with fewer than two slots): held.
- D7 audit × D4 (a passing entry fails on audit): held. It becomes an outcome entry.
- D8 × D9 (a hub with fewer than two slots, a pass ends with queued work): held.
- D8 × D9 (a no-argument hand call without `--upkeep` runs as an unregistered pass): held. D8 states the bound.
- D8 × D11 (a pass ended by an anchor move): held. The restart condition that started it still stands afterwards, with or without an exit-record write.
- D8 × D11 (a pass whose own registration fails): broken. See finding 1.
- D9 × D10 (yield per round, lock released and retaken, plan afresh): held.
- D9 hand call × pass (lock wait and slot hold on two-slot and three-slot hubs): held. The hand call waits at most one job, and there is no cycle because a pass holds one lock at a time.
- D10 × D11 (SIGTERM, then the finished rounds are written within `EndSparing`'s grace): held.
- D2 × D4 (an outcome or spawn-marked entry is no recorded test): held.
- D3 × D7 (the self-check reads the function sets of linked packages): held.
- D6 × D10 (the digest is taken before the `{parallel}` substitution; the tier-0 parallel is used at run time): held.
- D12 × D8 (only `--upkeep` calls read and write the exit record; the dobby reads its queued count): held.
- D13 Open risks × the "Added after Discussion" entry: broken. See finding 2.
- D8 acceptance criteria × D8's rule for hubs with fewer than two slots: broken. See finding 4.
- Constraints (PATTERN-gate-slot, config-layers, fabric-boundary, tolerant-records, daemon-wakeups) × the decisions: held. The PATTERN-child-processes constraint fails through finding 1.

**Findings**

1. **MEDIUM, design.** Location: D8 × D11 (and the PATTERN-child-processes constraint). D8 still admits a pass that never registered. Such a pass makes an anchor move wait for its whole unbounded run, and D11's end does not reach it, so D11's purpose fails.
   - D8 has the `--upkeep` verb register itself as `runproc.RoleTestMap` in place of the dobby's registration, and its Bound accepts "a pass started beside an unregistered one". The record does not say what the verb does when its registration fails.
   - The dobby still starts the pass as its own child. `dobbyPassInFlight` counts every live child of the dobby that the spare set does not cover. D11 only spares a *registered* `RoleTestMap` (`internal/fabriccli/anchordeps.go:459-490`).
   - `EndSparing` ends only processes that hold a record (`internal/runproc/end.go:55-67`).
   - So the anchor-move loop in `internal/fabriccli/anchor.go` polls `passesInFlight` for the whole pass. With D8's batch cap gone, that wait has no bound, which is exactly the case D11 removes.
   - Today's precedent is to leave a refresh unregistered when registration fails (`internal/beamcli/dobbydeps.go:605-620`, the "left to end on its own" Warn).
   - Fix: D8 makes a `--upkeep` call that cannot register itself exit before it plans.

2. **MEDIUM, consistency.** Location: Open risks, first bullet (decision-record.md:303-304), and support-log Q15. The record still asks the parent whether `testmap-cross-package`'s run is held.
   - The "Added after Discussion (parent, 2026-11-11)" entry (decision-record.md:278-282) already answers it: the run is paused at Run-Shed until this record is approved and D13's changes reach it.
   - The rubric classes an Open risks line that such an answer settles as consistency, and its fix drops the line.
   - Fix: drop the Open risks line and resolve Q15.

3. **LOW, consistency.** Location: D1's Current and Affected bullets (decision-record.md:34-37) × D4 (decision-record.md:102-104). D1 says every entry is kept under these verdicts; D4 drops outcome entries under them.
   - D1 says a Current verdict keeps every entry, and an Affected verdict keeps every unaffected entry.
   - D4 drops an outcome entry "when any of the package's own files differs between the map's `Commit` and `H`". Under D2, an own-file body change can still give a Current or Affected verdict, so D4 drops entries D1 says it keeps.
   - The D4 criterion ("a change to one of B's own files drops it") follows D4, not D1.
   - Fix: D1's bullets make an exception for D4's outcome-entry drops.

4. **LOW, consistency.** Location: the D8 acceptance criterion "with no landing since the last pass and an exit record naming queued work, the dobby starts a pass on its next tick" (decision-record.md:334-335). The criterion states no slot condition, but D8 starts no pass on queued work alone while the hub's slot count is below two.
   - The criterion at line 338 covers the one-slot case, so the two criteria contradict each other on such a hub.
   - Fix: qualify the line-334 criterion with "on a hub with two or more slots".

Tool calls: none. The inherited context held everything this lens needed.
