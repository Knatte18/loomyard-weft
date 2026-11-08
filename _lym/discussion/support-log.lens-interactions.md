```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions in the record that act on the same state, and ran each pair's failure and restart paths. Nothing is blocking. Four MEDIUM findings and three LOW findings follow.

**Findings**

1. **MEDIUM** — D2 × D4 × Raddle-File's return path: the record never says how the split perch runs again after Raddle-File sends the run back.
   - Evidence: in auto mode, Raddle-File's `sendBack` sends the run back to Raddle-Write (`raddlefile.go`, `raddleMaxReturns`). The next Raddle-Write session moves the proposal to the returned path and writes a new one.
   - With D4 in force, that new proposal reaches Raddle-File again only through the perch. The perch's Bouncer already settled "approved" earlier in the same run.
   - The record does not say whether the Bouncer reviews the new proposal from round one, how its bounce budget counts across returns, or whether a stale approved settle sends the run straight on to Raddle-File.
   - The note under "Notes for the plan writer" covers only whether the segment layout allows the return, not what the perch does on re-entry.
   - If the stale settle passes the run through, a re-proposed split is filed without review. That undoes D2 and D4's claim that filing rests on the perch's approval.
   - The fix is a decision that a return to Raddle-Write re-arms the perch with a fresh review, plus a bound on how the returns and the budget interact.

2. **MEDIUM** — D4 × D5: D4 and D5 name two different ways of telling that a run is an arras run.
   - Evidence: D4 decides auto mode "by the recipe's rows". D5 runs its outcome checks "when the run is an arras run" and puts them in `internal/raddleparser`.
   - The record does not say how the parser, the `raddle` gate that D2's fixer runs, `raddle-propose` and Raddle-File each learn that the run is an arras run.
   - If they learn it from the recipe name, D5 breaks the `PATTERN-shed-recipe-registry` constraint that D4 complies with.
   - If one of these check sites runs without the arras option, the fixer can change the shape to `beam`, or to `tasks` with several parts, and pass the gate.
   - The fix is one source for the signal, read from the recipe's rows (the presence of the perch) or recorded in the raddle state with the mode, passed to every check site.

3. **MEDIUM** — D6 × D8 × the run lock: the record does not say whether the converted project can be released while the arras run's batten rows still run.
   - Evidence: D8 says "A project's own run lock refuses the write as it does today". D6 clears the entry's run status in the filing write, but the arras batten run still has Land and its cleanup rows ahead of it under the same slug.
   - If the lock is keyed on status, the project can be released at once, before the arras run has landed its records.
   - If the lock is keyed on the run, the operator's release is refused until cleanup finishes, and the record names no way forward for that case.
   - The record should state which of the two holds, and the refusal's way forward if the second does.

4. **MEDIUM** — D6 × the run-status writes: the exemption D6 gives `SetRunStatus` sits on a path that pawl and beam projects use too.
   - Evidence: `battenCommitStatusDeps.SetBoardStatus` calls `board.SetRunStatus(runID, status)` with the slug for every row of the outer run, including the rows after Raddle-File such as Land. `pawlwiring.go` writes run status onto project entries legitimately.
   - D6 marks the exempt entry as "a project filed from a raddle run under its own slug". That rests on the body prefix that `isRaddleFiledFrom` reads, which is a prose test over the body.
   - If an operator later makes the project a beam (D6's `lym beam create` path), the beam's pawl run writes status onto the same slug, and the exemption would silently drop those writes.
   - This pair breaks unless the exemption ends once the arras run's records are cleared. The record's bound does not say that.

5. **LOW** — D7 × D5 and D6: the record is silent on a `decision add` after Raddle-File has filed.
   - Evidence: the run is still past its approval during Land, so `copyRecordToBoard` overwrites `decision-<slug>.md`.
   - In the family case that file is the project's stored record: the Sketch record after `MapPatternChanges`. It would be replaced by the worktree record, and no filed part would get the new entry.
   - In the one-task case the record is written onto a done entry that prune then removes.
   - D7's claim that "every filed part's record carries it" holds only before the filing write. The record should bound D7 to the rows before Raddle-File, or name what a later add does.

6. **LOW** — D6: the record does not say what happens to entries that depend on the arras entry.
   - Evidence: `MakeSketch` refuses when another entry depends on the entry, and the board's validation runs on conversion.
   - D6 turns the task into a project in the filing write. If an outside entry depends on the arras slug, the board may refuse the conversion, and the record does not say whether such a refusal returns the run to Raddle-Write or halts it.
   - The one-task path has no such gap, since prune strips dependencies on done entries.

7. **LOW** — D3 × D5: D3 has the rubric judge "the arras filing rule of D5". D5 already makes that rule a mechanical check, and D3 itself says mechanical checks are not re-derived. The fix is to drop that bullet, or narrow it to the judgement part, for example whether one task or a family fits the record.

**Pairs checked**

- D1 restart (the `undiscussed` gate) × D4 (mode recorded in the raddle state): held.
- D1 × D6 (no Sketch, since the recipe has a RaddleFile row): held.
- D1 × D7 (the past-approval test for an approval row inside a block): held if `PastDiscussionApproval` reads expanded rows, and the D7 acceptance criterion covers it.
- D2 × D4 (perch approval replaces operator approval): held on the first pass, broken on the return path (finding 1).
- D2 × D5 (the fixer's gate must enforce the outcomes): broken unless the signal reaches the gate (finding 2).
- D2 × D7 (the perch reads the record every round, and the overlay fixer edits only the proposal): held, with the residual window recorded under Open risks.
- D3 × D5: redundant (finding 7).
- D4 × D5: broken on the signal source (finding 2).
- D5 one-task × the landing prune: held.
- D5 family × D6 (the slug exemption, and the re-check exempting the run's own entry): held.
- D6 × `raddleFilingHeld` retry: held, since the converted body opens with the project prefix.
- D6 × `SetRunStatus` and later batten status writes: held for the arras run; open on later beam runs (finding 4).
- D6 × D8: open on the run lock (finding 3).
- D6 × entries depending on the arras entry: unstated (finding 6).
- D7 × D5 and D6 after filing: unstated (finding 5).
- D8 × dobby's `depends_on` gating: held.
- D8 × D9 (the policy names a project release as one release): held.

**Acceptance criteria and constraints**

- Every acceptance criterion matches its decision. D4 and D8 state their bounds, and their criteria test them.
- Constraints: `PATTERN-shed-recipe-registry` is at risk from D5 (finding 2). `PATTERN-sole-parsers` holds. The rest hold.

**Tool calls**

- I ran one combined Bash call that read `internal/battencli/commitstatus.go` lines 110–160 and grepped `KindProject` in `internal/boardengine/project.go`. My context had neither which end-of-run status writes batten makes on the slug, nor the project validation rules.
