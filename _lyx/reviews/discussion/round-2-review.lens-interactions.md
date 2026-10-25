```yaml
lens: interactions
outcome: reported
```

Restating my task: the interactions lens, which checks pairs of the record's own decisions against each other and then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D2 (Drift-Await ungated) × Added after Discussion F10 (Drift-Await behind `entry_gate: drift.enabled`): broken.
- D5 (a running capture holds Webster back even with the flag off) × F10: broken.
- D2 placement × Added after Discussion F9: held. Open risks still carries the question F9 settled: broken.
- D2 (no Drift acceptance under `full` before the parent rules) × D6 halts × D7 "caller at a halt" acceptance: broken.
- D3 scope verb × D7 accept/withdraw on the same scope record: held.
- D3 re-entry × D4 main-capture refusals (branch point, clean tree, current row): held.
- D3 re-entry × D5 adopt (no capture record on the adopting machine): broken, because the rule is silent on an absent record.
- D3 darn Stuck on a failed capture × D5 `lyx drift defer` (Drift-Await only): broken, because darn has no way to go on.
- D4 log-commit rule × D5 outcomes (Stuck, defer) × F10 skip: broken.
- D4 fresh capture record × D5 `awaited_at` in the scope record: held.
- D5 defer × D6 no-passed-capture halt: held internally; the goto it names is outside this lens.
- D5 HEAD-moved defer × D6 compare against the passed capture: held.
- D6 decision-halt set × D5 Drift-Await halts under `full`: held, since the later Drift-Verify decision halt bounds a deferral.
- D6 counter reset × re-prompts × resume: held.
- D6 handoff order × D7 skip-file acceptances: held.
- D6 resume after the parent's accept or withdraw × D7 attempt-tagged acceptances: held.
- D1 timeout × D5 unbounded wait: held.
- D1 undeclared-command refusal × F10 gate: held.
- Constraint "no row builds between Drift-Baseline and Drift-Await" × D4 slot sharing with the plan type load: held.
- Constraint "pairteardown ends the capture via EndSparing" × D4 role `capture`: held, since EndSparing spares only RoleDobby.
- AC1 (Drift-Await Done with the flag off) × F10: broken.
- AC "Drift-Await waits on a capture, a slot wait included" × D4/D5: held.
- AC "under full, an agent acceptance ends in a decision halt and the parent's accept then reaches Done" × D6/D7: held for the agent path, broken for the arbiter path (finding 2).
- AC on accept/withdraw refusals × D7: held.
- Support log Interview turn 3, Review rounds and Question ledger × the record's current D4/D5: broken.

**Findings**

1. **BLOCKING, consistency.** `decision-record.md` D2 (lines 72–73) and AC1 (line 229) contradict F10 (lines 199–203).
   - D2 says Drift-Await carries no entry gate and shows Done with the flag off. AC1 says a loom run records it as Done. The parent's F10 says it gets `entry_gate: drift.enabled` and records Skipped.
   - The support log's Rejected alternatives (line 317) and Q6 (line 341) still argue the overruled side.
   - The plan writer gets two opposite instructions. D2's paragraph and AC1 must follow F10.

2. **BLOCKING, design.** The arbiter can accept drift under scope `full`, which breaks D2's promise that the parent rules first.
   - D2 (lines 68–69) promises that no drift in a `full` task is accepted before the parent rules. D7 (lines 179, 182, 186) bounds `full` only through acceptances made by the agent session, which force a decision halt.
   - An acceptance by "a caller at a `Drift-Verify` halt" triggers no decision halt. Under `full`, D6's dirty-tree or test-gate Stuck (line 166) carries no `DecisionHaltPrefix`, since only the halts listed at line 170 do. That halt is a blocked stop the arbiter may settle (`internal/arbiterstop/arbiterstop.go:66-70`).
   - So the arbiter or tackler can run `lyx drift accept` there, then resume. The mechanical compare passes with the test skipped and the row returns Done, with no parent ruling.
   - D7's sentence "the arbiter under scope `partial`" names a bound that nothing enforces. Fix: every non-agent acceptance under `full` must also end in a decision halt, or `lyx drift accept` must refuse under `full` at any halt that is not a decision halt.

3. **MEDIUM, design.** F10's gate undoes D5's guarantee that a running capture holds Webster back (D2 line 72, F10 line 201).
   - D2's own reason for leaving Drift-Await ungated was that a capture still running when the flag is turned off must hold Webster back.
   - Under F10, Drift-Await is Skipped with the flag off, so Webster builds beside a running capture, in the same worktree and against the same baseline directory. The record says nothing about this case.
   - It must choose: ending the capture when the gate skips, a refusal on flipping the flag while a capture runs, or accepting the overlap with a stated bound.

4. **MEDIUM, design.** Darn's failed-capture halt has no way forward that goes on without regression (D3 line 92 × D5 line 136).
   - In darn, Drift-Baseline is Stuck on a failed capture, but D3 names no way forward for that halt.
   - D5's `lyx drift defer` refuses unless the run is halted at Drift-Await, which darn lacks.
   - A darn task whose harness keeps failing can only re-enter and recapture, so it is stranded. This also breaks the Constraint that every halt names its way forward (line 209).

5. **LOW, consistency.** D3's re-entry rule (lines 95–97) does not cover a missing capture record, which D5's darn adopt path (line 143) relies on.
   - D3 starts a capture "only when none holds the capture lock and the last one did not pass". It says nothing about no record at all, which is what an adopting machine has, since `.lyx/` is not carried.
   - D5 depends on D3 capturing again in that case. D3 should state that an absent capture record counts as not passed.

6. **LOW, design.** D4's rule for committing the capture log (line 119) does not match D5's outcomes (lines 132–137).
   - D4 says Drift-Await commits the loom capture log. D5 commits it only on the passed-await Done.
   - Neither covers a Stuck halt that names the log, a `lyx drift defer` Done, or a Skipped Drift-Await under F10.
   - In each of those cases, a log the halt points at goes uncommitted. Each Drift-Await way out should commit the log, or D4 should name who commits it.

7. **LOW, consistency.** `support-log.md` contradicts the record and itself.
   - Line 281 records the fixed await bound "timeout plus one minute… then Stuck", which D5 line 130 replaced with no bound of its own.
   - Line 291 records "a new child entry, PATTERN change listed", which Q12 (lines 348–349) superseded.
   - Line 332 reads "_No rounds yet._" while Q11–Q15 cite review round 1.
   - Turn 3 should mark the superseded fixes, and Review rounds should record round 1.

8. **LOW, consistency.** Open risks (lines 222–223) still raises Drift-Verify's position to the parent, which F9 (lines 193–197) answered. Per the rubric, the fix drops the line.

Tool calls: none.
