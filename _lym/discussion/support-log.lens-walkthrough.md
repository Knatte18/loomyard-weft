```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked whether it meets its own purpose.

**Findings**

- **BLOCKING — D1 / D6 (plan-conflict reroute):** a Weaver plan-conflict reroute lands on a Plan-Burler-Review that cannot see the conflict.
  - Case: a batch's begin refuses the approved plan, writes its conflict record and takes `on_reroute` to Plan-Burler-Review.
  - The next step reads a different path from the one Weaver writes. The Plan-Review row's prompt in `contracts/recipes/tapestry-recipe.yaml` names `.lym/webster/plan-conflict.md` by its literal path.
  - The same prompt says to fix only "cards Webster's auto-rebaseline accepts: a pending card, or a card of a failed or in-flight batch", which it judges from Webster's state.
  - The reroute's readers are `shedadapters/webster.go` and `websterengine/conflictrecord.go`.
  - D9 puts Weaver's scratch under `.lym/weaver/`, so the review round finds no conflict and the reroute cannot settle it.
  - The record must say where Weaver's conflict record lives and how the recipe's review text and the route claim read it for either implementer. D11 lists the plan-conflict refusal as shared machinery but does not cover its reader in the recipe prompt.
- **MEDIUM — D10 (the row's gates entries):** "runs the row's own `gates` entries" hands Weaver a shape it cannot run as a Go check.
  - `internal/shedrecipe/entries_gate.go` `resolveRowGateSpec` turns the row's `gates` key into a `shuttleengine.GateSpec`.
  - A GateSpec is a gate on a strand that runs at turn ends and re-prompts that strand. The drift entry with `attempts: 3` re-prompts Merriam today.
  - Weaver has no long-lived strand. After the verify, the drift entry has no strand to gate and no agent to re-prompt with its attempts.
  - The decision must say which strand carries the entries (each batch strand, the fixer, or both) or how Go runs them without one.
- **MEDIUM — D7 (re-entry after an arbiter fix):** re-entry skips the post-batch checks, so the arbiter's fix never binds handles or applies renames.
  - Case: the arbiter commits a fix that renames a glyph, then gotos the row.
  - Weaver runs only the card gates at HEAD, they pass, and the batch is recorded done.
  - The checks D11 lists (done checks, handle binding, scope guard, and renames rewriting pending cards) never run for that batch.
  - The next D3 partition is then built from cards that still name the old glyph, and created handles stay unbound. D3's purpose, partitioning from the rewritten plan, is missed on this path.
  - Re-entry should run the full post-batch checks, not the card gates alone. The arbiter's fix has no report and so no `renames` field, which the record should also address.
- **LOW — D9 (commit after every batch):** "Go commits `_lym/weaver/`" names no committer.
  - `_lym/` is fabric-synced weft content, and PATTERN-fabric-boundary puts every git run on the weft in `internal/fabricengine`.
  - The commit must go through fabric's records commit. The Constraints section omits that entry.

**Decisions walked**

- D1, case `run_settings` `tapestry.implementer: weaver`: the row builds and the edges are unchanged. Purpose met, apart from the reroute reader above.
- D2, case: batch 2 exhausts the ladder, goes Stuck, the arbiter fixes and gotos the Webster row. The re-step re-enters Weaver. Purpose met (re-entry gaps are under D7).
- D3, case: three cards, with card 1 done and reporting a rename. The cost batcher runs over cards 2 and 3 as rewritten. A boundary that depends on a done card only splits batches among the done cards, so nothing is lost. Purpose met.
- D4, case: first attempt on `sonnet[medium]`. The prompt gets the cards, the record and the journal, and the strand's output file is the report. Purpose met.
- D5, case: attempt 1 fails, attempt 2 runs on `opus[medium]`, attempt 3 fails, and the row goes Stuck. Purpose met.
- D6, case: a `NEEDS_PARENT` report. The row goes Stuck with the need, and re-entry follows D7. Purpose met.
- D7: purpose missed (finding above).
- D8, case: an attempt dies with no report. No journal entry is written, as designed. Purpose met.
- D9: committer unspecified (finding above).
- D10: gates entries cannot run as described (finding above).
- D11: the shared pieces have their inputs, except the plan-conflict reader (BLOCKING finding).
- D12, case: a crash mid-attempt. On the next call Weaver attaches to the live strand, or counts a gone strand with no report as a failed attempt. Purpose met.

**Tool calls**

- Read the tapestry recipe's Plan-Review text and grepped for the plan-conflict readers. My context did not show how the reroute's review round finds the conflict record.
- Grepped `loomshed/webster.go` and `shedrecipe` for how the row's `gates` key is consumed. My context did not say what shape the gates entries take.
