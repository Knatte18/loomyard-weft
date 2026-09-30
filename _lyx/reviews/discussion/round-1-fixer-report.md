# Fixer report — Discussion-Review round 1

Every finding in `round-1-review.md` is fixed.
`lyx loom validate-discussion` exits 0 on the result.

## Fixed

- **F1 (BLOCKING):** D5 and D8 now name record-batch as the only per-batch audit site, and D8's rationale states that recover-batch runs no audit.
  D8 adds the run-exit decision: the whole-session cross-check applies the same classification, passes findings already dispositioned in `state.json`, and hard-errors only on undispositioned ones.
  The named-spawn must-cover scenario now asserts that the run-exit cross-check passes.
- **F2 (BLOCKING):** D4 now lists the correctness class first, covering the warp's tracked tree and anywhere under the run's `_lyx` content, and states that correctness wins.
  The rationale is rewritten to match.
  A must-cover scenario is added for a Master write to `_lyx/webster/state.json`.
- **F3 (MEDIUM):** D11 decides the budget reset.
  A history-only `goto` outcome is an episode boundary for every producer sharing the target's `Segment`, and loom's seed-coherence check accepts it.
  A new Constraints bullet records the history-contract change, and the exhausted-budget scenario now says "every producer in the segment".
- **F4 (MEDIUM):** D12's invariant now binds only the modules that have a section in `refusal-spec.md`, and other modules join through their own audits.
- **F5 (MEDIUM):** D11 now decides entry-gate re-entry: goto places no restriction on targets, and `Loom-Preflight`'s half-finished check is a table row whose way forward is `goto` the next row or seeding a new run.
  The open risk is removed, support-log ledger item 17 is resolved, the Notes line is updated, and a must-cover scenario is added.
- **F6 (MEDIUM):** D6's digest names each correctness-class `parent-write` path, and the recovery strand treats those paths as suspect.
  The remaining exposure is recorded as an open risk.
- **F7 (MEDIUM):** D10 now refuses when any begun or terminal batch covers a different card set under the edited plan.
  The rebaseline scenario now includes regrouping.
- **F8 (LOW):** D5 now defines a failed verify (it fails to start, exits non-zero or times out) and sends a failure before any verify command starts to D2's grouped transient row.
- **F9 (LOW):** goto now writes state `paused`, with the rationale in D11.
  A must-cover scenario checks that the next loom run files no crash-resume anomaly.
- **F10 (LOW):** The per-row test criterion is split into halting rows and policy rows, and "progresses" is replaced by "the refused verb, or the verb the way forward names, succeeds".
- **F11 (NIT):** Each Auto-mode assumption now names only its self-pick and points at the decision.

The support log gains a Round 1 entry under Review rounds and five rejected alternatives from this round.
Ledger item 7 is revised, and rows 19–21 are added for the questions this round settled.

## Deferred

None.
