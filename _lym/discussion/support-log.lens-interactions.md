```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then checked the acceptance criteria and constraints against the decisions they touch.

**Findings**

1. **BLOCKING — D4 × D6 × D3 × D5: `lym loom decision add` always fails on a darn record.**
   - D4 writes the record with no support log.
   - `decision add` appends through `discussionparser.AppendDecisions(recordPath, supportLogPath, ds)`. That calls `discussionparser.Validate`, which returns a `discussion-file-missing` finding as soon as the support log is absent (`internal/discussionparser/validate.go:66-70`).
   - On a finding, AppendDecisions puts the prior bytes back and nothing lands (`internal/discussionparser/append.go` doc, call at line 93).
   - D5 drops the support-log check only for "the one gate and validator" in loomshed. It says nothing about the parser-level `Validate` inside AppendDecisions.
   - Result: D6's answer route through the parent's `decision add`, D3's `--pattern-*` on darn, and the D5 acceptance about keeping Added entries all fail on darn.
   - The fix has to cover the append path's validation too, or D4 has to write a support log.

2. **MEDIUM — D5 × D6: a darn writer with no parent can no longer halt the run.**
   - Today the no-parent stop works because the writer's `discussion` gate exhausts on a record that fails validation. The darn stencil says to "end your turn with the stop report unchanged, so the gate exhausts and halts the run".
   - Under D5 that gate only checks that the mechanically written record is intact, so it passes.
   - The stop report sits in the change description, and the run carries on through verify, Drift-Verify, Code-Review, Describe and Finalize. D6 says "stops with a stop report… as today" but nothing halts it any more.
   - D6 needs a halt mechanism of its own, or its acceptance criterion should say how the stop halts.

3. **MEDIUM — D4 × D5 × D9-style resume-by-name: darn runs already in flight hit a missing record.**
   - A darn run parked before Darn-Write (at Preflight, Drift-Baseline or Drift-Await) under the old recipe has no record. It resumes by row name past the new Record-Write row, which sits before those rows.
   - The new writer stencil no longer writes a record, so the validate-only `discussion` gate fails on the missing record, exhausts, and halts the run.
   - The record is silent on this. It needs either a retired-row or edge note that sends such runs to Record-Write, or a statement that they are restarted.

4. **MEDIUM — D8 × D2: pawl's verify source is left undefined.**
   - D8 swaps the recipe-name branch for a recipe-file `verify_source` key. Today pawl takes `planVerifySource` as the non-darn default (`internal/loomcli/arm.go` `verifySource`; "A beam run… runs the `finalize` gate key over the whole merged tree").
   - `pawl-recipe.yaml` declares no contract and, under D8, no `verify_source`. The record doesn't say what pawl reads, or whether D2's exception also covers this branch.
   - Relatedly, D1 doesn't say whether `verify_source` is required for a `contract: loom` recipe, or what a recipe that omits it defaults to.

5. **LOW — D3 × D6 acceptance: no criterion checks that darn's writer applies PATTERN bullets.**
   - D3 says darn's writer "reads the record's `## PATTERN changes` and applies them".
   - The D3 acceptance criterion only checks that `--pattern-*` appends the bullet, and the D6 criterion covers only the outdated-entry route.

**Pairs checked**
- D1 × D4 (record engine set includes RecordWrite; Record-Write before Darn-Write): held
- D1 × D9 (writer before Finalize; Finalize on_done goes to the terminal): held
- D1 × D10 (Tapestry-Preflight removed; contract unaffected): held
- D1 × D8 (`verify_source` not covered by the contract check): open, see finding 4
- D2 × D14 (refusal names the `lym shed` verbs, which still serve every recipe): held
- D2 × D8 (pawl's branches: verify source left unspecified): broken, finding 4
- D2 × D3 (peers read a pawl peer's own recipe): held
- D3 × D8 (applicability keyed on rows; `--amends-plan` keyed on Plan-Write): held
- D3 × D4/D5 (`--pattern-*` append on darn): broken, finding 1
- D4 × D5 (commit, then history check; writer gate is validate-only): held for new runs
- D4 × D5 × resume (in-flight darn runs with no record): broken, finding 3
- D4 × D6 (answer via `decision add`): broken, finding 1
- D4 × commit pathspec (pathspec is the discussion dir, so a missing support log is harmless): held
- D4 × `seed-starts-at-start` (seed still names Preflight, which stays the entry row): held
- D5 × D6 (no-parent stop relied on gate exhaustion): broken, finding 2
- D5 × D7 (answer key is the record; history check keeps Added entries): held
- D5 × `validate-discussion` (checks follow the record row's engine; lightweight wiring reads the recipe): held
- D5 history check × tapestry Discussion-Write first gate (no history yet): held
- D6 × D7 (Added entries amend the answer key): held
- D9 × Finalize `on_reroute` to Parent-Align (a reroute now passes through Drift-Verify again before the perch): held, and arguably intended
- D9 × resume of a run parked on Drift-Verify after its review (perch runs again, as stated): held
- D10 × D1 (batcher load keyed on the Implementer row, which is also in the writer set): held
- D11 × D2 (pawl seeds record `pawl` since the rename): held, resting on the entry's claim
- D12 × D13 (`driver` model stays, `driver_kind` retired; KeyMoves vs retired keys are separate mechanisms): held
- D12 × recipe setting references and review budgets (`loom.plan_*` references renamed; budget reads follow): held, as a plan-level detail
- D13 × the PATTERN-shed-stop-chain change: held
- Acceptance D3, D4, D5 and D6 against D4's missing support log: D3's `--pattern-*` criterion and D5's "refused on both recipes" for dropped entries can't be exercised through `decision add`, see finding 1
- Constraint PATTERN-gate-slot and parity × D5: held

**Tool calls**
- Read `discussionparser/append.go` and `commitDiscussion`: my context didn't show whether `decision add`'s append validates the support log.
- Grepped `discussionparser/validate.go` for its support-log check: to confirm Validate refuses when the support log is missing.
- Read `discussionCommitPathspec`: to check whether committing without a support log would break the shared discussion commit.
