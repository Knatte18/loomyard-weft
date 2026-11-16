```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked the record's bounds, guards and claims about existing code. Two claims break outright (D7, D10), four are weaker than stated (D12, D1, D5, D3), and the rest hold.

**Findings**

1. `BLOCKING`, D7: a stopped batch can be recorded done without its work being built.
   - **Counterexample:** batch 3 stops on `NEEDS_PARENT` before any commit. Its card's Verify line runs a package tier that already passes, which is common when the card adds code and its tests together. The arbiter answers with a `decision add` and a goto with no code fix. On re-entry the card gates pass at HEAD and the tree is clean, so D7 records the batch done.
   - **What it skips:** `index.DoneChecks`, called in `postBatchChecks` (`internal/websterengine/recordbatch.go`), which checks the batch's targets. It also skips the scope guard and rename handling, which D11 says every batch gets.
   - **Fix:** on re-entry, run the full post-batch checks against the batch's recorded start commit, not the card gates alone.

2. `MEDIUM`, D10: the row's `gates` entries have no runner once there is no strand.
   - **Evidence:** the row's `gates` key goes through `resolveGateSpec` into a shuttle gate spec attached to Merriam's strand, and Webster runs it at that strand's turn ends (`internal/shedrecipe/entries_simple.go`, `internal/websterengine/runlevel.go`). That constructor also rejects unknown keys (`configRejectUnknown(cfg, "gates")`).
   - **Break:** D10 runs the drift entry in Go after the verify. That turns a strand gate into a Go call, and the record names no mechanism for it.
   - **Fix:** name the mechanism. Either attach the entries to each batch or fixer strand as shuttle gates, or have Go invoke the gate entry. Also state that Weaver's constructor accepts and handles the `gates` key.

3. `MEDIUM`, D12: two agents can run at once in one tree.
   - **Counterexample:** the producer calls Start, then the lym process crashes before state records the strand. On the next call state shows no strand, so Weaver spawns a second agent while the first is still live.
   - **Webster's guard:** `RecoverSpawnOrAttach` finds a run it did not record through `shuttleengine.FindRun`.
   - **Fix:** record the attempt before spawning, or look up a live run for the attempt's role before spawning. The run lock does not help, because the first holder is dead.

4. `MEDIUM`, D1: switching the implementer mid-run is unguarded.
   - **Counterexample:** a run stops in Webster after batches 1–2 are committed. The parent sets `tapestry.implementer: weaver` through `run_settings` and re-steps. Weaver's state is empty, so D3 treats every card as remaining and spawns agents over code that is already built. The reverse switch hits Webster's recorded partition and fingerprint against Weaver's commits.
   - **Fix:** state the guard. Either refuse, with a way forward, when the other implementer's run directory exists, or record which cards are done from that state.

5. `MEDIUM`, D5 and D6: the bound of one attempt per rung counts infrastructure faults as model failures.
   - **Counterexample:** a reboot, a hibernate or a reed loss kills the strand with no report. D12 counts that as a failed attempt, so the batch climbs a rung, and on the last rung the row stops for the arbiter. The model never failed.
   - **Note:** shuttle already separates respawn-eligible verdicts and API-error retries.
   - **Fix:** state that a strand lost with no report gets a fresh attempt on the same rung, bounded, and that only a model-side failure climbs.

6. `LOW`, D3: re-partitioning loses a batcher boundary rule.
   - **Evidence:** `boundary.go` forces a new batch when a card renames or deletes an earlier card's target. Webster sees every card because it partitions once.
   - **Break:** Weaver partitions only the remaining cards, so it cannot see a done card's rename. Rename reports rewrite pending cards, which covers most cases, but a delete is not reported as a rename.
   - **Fix:** pass the done cards to the batcher as context, or state that renames and deletes are settled through the rewrite.

7. `LOW`, D2 and D1: the record contradicts the contract it says holds unchanged.
   - **Evidence:** D2 says any other Stuck goes to shed's stop chain. `contracts/specs/implementer-spec.md` says the row has no `on_stuck`, "so the run halts for a human". The driver does send a blocked stop to the tackler first.
   - **Fix:** add the spec's Stuck wording to D1's spec edit.

8. `LOW`, D4: the claim that "any engine" may run an attempt, "Codex included", has not been checked against what Codex sessions get.
   - **Evidence:** the board note `codex-same-as-claude` says a Codex session does not yet get the same skills, guards and instruction files as a Claude session. The record also does not show that gated re-prompts work on a Codex strand.
   - **Fix:** narrow the claim or bound it.

**Claims that held**

- D2: the arbiter's spawn cap bounds re-entries. `arbiter_max_spawns` counts per run and is read from disk, so a restart does not reset it.
- D9: committing only `_lym/weaver/` while a failed attempt leaves code uncommitted. The arbiter's snapshot is taken at its spawn, so leftover changes from before do not trip its precondition.
- D11: adding `journal` to the strict report parser. It holds provided the field is added to the shared contract, as D11 says.
- D6: a plan conflict takes the reroute, as through Webster's `ClaimConflictRoute`.
- D10: reusing the `webster` gate key.

**Tool calls**

1. Grepped `internal/sheddriver` for `arbiter_max_spawns`, `internal/websterengine`, `internal/loomshed` and `internal/shedadapters` for the drift entry, and `recordbatch.go` for `postBatchChecks`, to confirm whether the arbiter cap counts per run and what the post-batch checks contain; my context had only the subagents' summaries.
2. Grepped `internal/loomshed/webster.go` and `runlevel.go` for how the row's gates reach Webster, to test D10's mechanism.
3. Grepped `internal/shedadapters` and `internal/shedrecipe/entries_simple.go` for `resolveGateSpec`, which showed the row's gates are a shuttle gate spec.
4. Read the doc comments of `internal/sheddriver/driverules.go`, to check the route a Stuck stop takes against the implementer spec's wording.
