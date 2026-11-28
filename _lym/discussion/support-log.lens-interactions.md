```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked the pairs of decisions that act on the same state, then each acceptance criterion against the decisions it touches.

**Findings**

1. **BLOCKING — D5 with D2 and D3: the start refusal also catches undiscussed tapestry runs.**
   - D5's predicate is true when a recipe's "expanded rows hold a RecordWrite row with `stored_only` true".
   - Under D2 and D3, tapestry's expanded rows inline the Jacquard block, and that includes jacquard's stored-only Record-Write row (D4).
   - So the predicate is true for tapestry. Batten's start guard and `lym loom start` would then refuse every undiscussed tapestry entry, which is the case tapestry exists for.
   - The criterion "the predicate is … false for tapestry" contradicts D5 as written.
   - D5's way forward ("run the entry as tapestry") would then loop back to the same refusal.
   - Fix: make the predicate also require that no record-producing row (DiscussionWrite) comes before the stored-only row. Alternatively, read only the recipe's own top-level rows.

2. **MEDIUM — D6: its filing bound is broken by the code.**
   - D6 says the check "never refuses a filing of a proposal approved before this change".
   - `raddleFile.verify` in `internal/loomshed/raddlefile.go` runs `p.deps.Check()` at filing time. Any finding sends the proposal back with "the proposal fails its checks".
   - So a proposal approved under the old vocabulary, naming a tapestry part and filed after the upgrade, is sent back.
   - In interactive mode the proposal's hash is also matched against the approval, so editing the part to jacquard forces a new approval.
   - Fix: either make the filing-time check accept tapestry for a proposal approved before the change, or state the send-back and re-approval as the bound.

3. **MEDIUM — D4 with D5: the Stuck's first way forward does not move a run that has already started.**
   - D4's Stuck reason offers `lym board upsert … "recipe":"tapestry"`.
   - A jacquard run that reaches Record-Write has already written its seed, and the seed's recipe is recorded. `sharedbootstrap` resolves the recipe from the existing seed.
   - Changing the board entry's recipe therefore does not change the running recipe, and resuming lands on the same Stuck.
   - That way forward needs the run cancelled first (`lym batten cancel &lt;slug&gt; --remove`), then a release and `lym batten run`, as Record-Write's existing error-envelope reason spells out.

4. **LOW — D2 with D4: the goto way forward is not proven.**
   - D4's way forward for a tapestry run parked at Preflight is `lym loom goto --to Motif`.
   - Motif becomes an engine-Block row after expansion. Whether goto admits a block row as its target, and whether the block gate then runs, is not stated and not covered by a test.
   - The refusal-way-forward pattern needs that way forward reached by a test.
   - Fix: add it to the D4 criterion, or name a goto to Discussion-Write.

5. **LOW — D8 with D2: no decision covers the phase table for tapestry's new rows.**
   - The tapestry phase table classifies the Motif row as Discussion.
   - Under D2 a tapestry run gains the Jacquard block row and runs Preflight and Record-Write after Discussion. No decision says which phase these fall under in tapestry's table.
   - D8 only defines jacquard's table as "tapestry's without its Discussion phase".

6. **LOW — D9 with D6: the tapestry stored-record cost cell is emptied.**
   - Under D9, tapestry runs with `stored_record` true are counted only in the jacquard cell.
   - The example cell in the raddle spec is `{"recipe":"tapestry","stored_record":true}`, and stored proposals may name it. Such cells become empty.
   - This holds once D6 renames the cells to jacquard. State it as part of D9's bound.

**Pairs checked**
- D1 and D2 (top-level keys duplicated, equality pinned by a criterion): held.
- D2 and D3 (Jacquard as the last block with no `on_done`; Friction-Reflect ends the run; Discussion-Bouncer still takes Motif's exit): held.
- D2 and D4, undiscussed tapestry (the approved record exists, so Record-Write writes nothing): held.
- D2 and D4, discussed tapestry (the Motif gate seeds the record and support log, then Record-Write writes nothing): held.
- D2 and D4, run parked at Preflight with no record: held for correctness, with the goto gap in finding 4.
- D4 and D5 (Stuck and refusal share one row config): broken by finding 1, and the Stuck's way forward is broken by finding 3.
- D5 and D2/D3 (predicate over expanded rows): broken, finding 1.
- D5 and D6 (filed parts carry the discussed flag and record, so they pass the start refusal): held.
- D6 and the existing filing path: broken, finding 2.
- D7 and D2 (a tapestry run reads its tapestry block, a jacquard run its jacquard block; raddle's profile check passes): held.
- D8 and D5 (batten admits jacquard and the guard judges it): held.
- D8 and D9 (report selection and cost cells): held, with finding 6.
- D10 and D11 (glossary and nickname wording agree): held.
- Each criterion against D1 to D11: the D5 criterion contradicts D5's text (finding 1). The others match their decisions.

**Tool calls**
- Grepped `raddlefile.go` for a check call and `loomcli` for EngineBlock. My context did not say whether filing re-runs the proposal check.
- Read `raddlefile.go` lines 255–285, to confirm that `verify` runs `p.deps.Check()` at filing.
