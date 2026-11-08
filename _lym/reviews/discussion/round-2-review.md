---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:229
    summary: With the auto-accept key off, the Bouncer's soft-cap rule still accepts a spent-budget round whose open findings are not tagged design-wrong, and since D2 sets no carry_over those findings vanish. D4's Bound omits this route and the D2 criterion contradicts the code.
    fix: In D4's Bound (decision-record.md lines 90-94), name the soft-cap rule's automatic accept as a second route. With the budget spent, a round whose open findings carry no design-wrong tag is accepted and filed whatever the auto-accept key says (shedadapters/bouncer.go softCapRule). State its bound, either by keeping carry_over off and saying the open findings of such a round are dropped, or by naming where they are kept, such as the parent notice or the filed parts' Open risks. Rewrite the criterion on line 229 to match, for example that with the key off a spent budget whose open findings include a design-wrong one escalates and never reaches Raddle-File, and add a criterion for the soft-cap accept route.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:134
    summary: The filing mark is written after the board write, so a crash or failed store write in between lets loom's resume hook write a run status onto the converted project. The retry writes only the store, so that status stays, the project stays run-held and D8 refuses its release.
    fix: In D6 (lines 134-138), order the mark so this window cannot leave a status. Either write the filing mark before the board write and remove it when Raddle-File sends the proposal back, or state that a Raddle-File retry that finds the converted entry as filed writes the mark and clears any run status on the entry in the same step. Add an acceptance criterion that a family filing whose store write fails after the board write, and then resumes, leaves the converted project with no status and releasable by D8.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:62
    summary: D2's re-arm clears only the settle record. A Bouncer with a Subject seam keeps a generation record, so a changed proposal starts an amendment generation capped at the amendment allowance, not a fresh cycle with the full budget. D2 also never says whether the perch has a Subject seam.
    fix: In D2's re-arm bullet (lines 62-63), state whether the split perch's Bouncer gets a Subject seam. With one, say that the re-arm clears both the settle record and the generation record, or that the perch sets FullGeneration, so the new proposal gets the segment's full budget. Without one, drop the settle-record clear and say the Bouncer's existing clear on re-entry gives the fresh cycle. Keep the D2 criterion on line 228 and have it assert the full budget.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:152
    summary: D7's way forward before the filing is a goto into Raddle-Write, but D2 re-arms the perch and D4 sets the proposal aside only after a Raddle-File return. A re-split carrying the newly recorded decision can then settle on the old approval and file unreviewed.
    fix: In D2's re-arm bullet (line 62) and D4's set-aside sentence (line 88), replace "a return from Raddle-File" with every fresh entry into Raddle-Write, a return from Raddle-File or a goto into the row, as freshRaddleEntry already judges. Extend the D2 criterion on line 228 to read that after a Raddle-File return or a goto to Raddle-Write, the new proposal is reviewed by the perch before Raddle-File runs.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:123
    summary: D6's admission compares the entry against a project derived from the entry before the write. That entry cannot yield the part list the body and order must hold, and the Bound does not say the admission is open to any writer at any row.
    fix: In D6's admission (lines 123-132), say where the run lock reads the filed parts. They are the entries the same write adds whose project is the slug, each a discussed draft task, and order and the body's part list must equal them in dependency order. Add to the Bound that the shape check is the only gate, with no row or writer limit, so a write of that exact shape during any row also passes and ends the run's hold. Either accept that, saying why, or limit the admission to an entry whose run status names the Raddle-File producer.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:218
    summary: The Open risk says motif-before-tapestry is in Code-Burler-Review, but it landed on main as c8e43ac4 and its board entry is pruned. That commit is not yet in this branch.
    fix: Drop the first Open risks bullet (line 218) and the matching Open line of the support-log Question ledger (support-log.md line 82). In D9 (line 180), add that the board-policy rule it rewrites is the Motif-first rule commit c8e43ac4 landed on main, so the run takes main's text of plugins/ly/skills/board-policy/SKILL.md and contracts/orchrole/orch-role.md before editing them.
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:136
    summary: D6 argues a rejected alternative, keying the status skip on the raddle store, inside the decision record, which the compaction rule keeps for Decision and Rationale only.
    fix: Delete the sentence on decision-record.md line 136 that begins The board's raddle store. Add a bullet to support-log.md's Rejected alternatives saying the post-filing status skip is not keyed on the board's raddle store under raddle/<slug>/, since no run clearing removes it and a later beam run under the slug would lose its status writes.
  - id: F8
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:43
    summary: D1 and the last Constraints bullet both state that arras holds no Record-Write row because the loom contract allows one record row.
    fix: Delete the last bullet of Constraints (decision-record.md line 207) and keep D1's sentence on line 43, which already cites the reason.
---

### [BLOCKING] The soft-cap rule files a spent-budget proposal with the auto-accept key off

**Location:** `_lym/discussion/decision-record.md:229`, D4's Bound (lines 90-94), D2 (line 56)

**Issue:** `settleUndecided` sends a spent-budget CONTINUE round with AutoAccept off to `softCapRule` (`internal/shedadapters/bouncer.go`, around line 1358).
When the round's ledger holds no design-wrong entry, `softCapRule` records an automatic accept and calls `acceptRound`, so the Bouncer returns Done and Raddle-File files.
The rule's own bound is the carry-over of open findings into `## Open risks`, "which the recipe sets".
D2 sets no `carry_over`, so a round with open MEDIUM findings tagged mechanical files with those findings dropped and no approval.
The record's criterion that a spent budget with the key off never reaches Raddle-File is false against the code, and D4's Bound names only the auto-accept route.
That breaks the rubric's attack-surface rule: this routing edge lets an unreviewed proposal through and the record gives no bound for it.
The handler reached the same reading on its own.

### [BLOCKING] A run status written between the filing write and the filing mark is never cleared

**Location:** `_lym/discussion/decision-record.md:134`, D6 (lines 134-138)

**Issue:** `FileRaddle` saves board.json and then writes the store.
A store write that fails after the save returns an error with `saved` true (`internal/boardengine/raddlefile.go:121-130`), and a crash can land at the same point.
D6 writes the mark only "when its board write lands with a project", so no mark exists yet.
On the resume, loom's status hook starts with an empty `lastBoardStatus` and calls `SetRunStatus`, which writes onto any entry that is not done (`internal/boardengine/board.go` `SetRunStatus`, `internal/loomcli/wiring.go:320-326`).
The run lock never judges an entry with no status (`internal/boardengine/runlock.go:159`), so the run status lands on the converted project.
The retry D6 describes "writes only the store".
The family outcome has no `EndRaddleEntry`, and the Done transition writes no board status, so the status stays.
`RunHolds` then holds the project and its members, and D8 refuses the release as a project with a run status.
That is the operator's one remaining step.
Four lenses (interactions, existing-paths, bound-breaker, walkthrough) found this independently.
It also settles the focus directive's second question: the key is the right run-scoped record, and `ClearMotifRun`'s extension ends the skip for a later beam run, but the write order leaves this window.

### [MEDIUM] Clearing the settle record does not give the re-armed perch its full budget

**Location:** `_lym/discussion/decision-record.md:62`

**Issue:** The Bouncer's re-entry tells four cases apart (`internal/shedadapters/bouncer.go:347-356`).
A Bouncer with a Subject seam keeps a generation record as well as the settle record.
With only the settle record cleared, case 3 meets a live subject that differs from the judge subject and starts a new generation through `startNextGeneration`.
That generation is an amendment unless `FullGeneration` is set (`internal/shedadapters/amendment.go`).
`budgetReached` caps an amendment's budget at `AmendAllowance` (`bouncer.go:1486-1488`), so the re-armed cycle does not get "the segment's full budget" D2 promises.
A Bouncer without a Subject seam keeps no settle record at all, so the mechanism D2 states does not apply.

### [MEDIUM] A goto into Raddle-Write does not re-arm the perch

**Location:** `_lym/discussion/decision-record.md:152`, D2 (line 62), D4 (line 88)

**Issue:** D7 sends an operator decision recorded at Raddle-File, before the filing, to `lym loom goto --to Raddle-Write` so the split is redone with the decision in place.
D2 re-arms the perch, and D4 sets the last proposal aside, only on "a return from Raddle-File".
`freshRaddleEntry` already treats a goto routed to Raddle-Write as a fresh entry (`internal/loomcli/raddlewiring.go:57-66`), but the record limits its new behaviour to the return.
With the old settle state in force, the re-split proposal can settle on the earlier approval and reach Raddle-File without review, which defeats D7's purpose.

### [MEDIUM] D6's admission names a derivation the before-entry cannot supply, and leaves the writer and row open

**Location:** `_lym/discussion/decision-record.md:123`

**Issue:** `endsAsSketch` can compare `now` against `sketchOf(before)` because a Sketch is a function of the entry alone.
D6's arras project is not: its `order` and its body's part list name the filed parts, and `before` holds none of them.
The record says the lock admits the entry only when it "equals the arras project derived from the entry before it", with order "exactly the filed parts".
It never says where the lock reads those parts.
A lock that reads them from `now.Order` would check nothing, so the bound the focus directive asks about is not stated.
Like `endsAsSketch`, the check sees only `before` and `after` (`internal/boardengine/runlock.go:163-166`).
A board write of that exact shape from any writer, at any row, passes and ends the run's hold mid-run.
The Bound says the admission "lets through only that change" but not that the change is open to any writer at any row.
The bound-breaker lens raised the writer-and-row half as LOW; the handler raises it with the derivation gap as one MEDIUM finding.

### [MEDIUM] The motif-before-tapestry Open risk is outdated

**Location:** `_lym/discussion/decision-record.md:218`

**Issue:** `lym board get` returns no entry for `motif-before-tapestry`, and `git log main` shows commit c8e43ac4, "Ship the rule: a tapestry-sized note runs as a Motif first", landed today at 13:36.
`git merge-base --is-ancestor c8e43ac4 HEAD` reports that this branch does not yet hold it.
The Open risk still describes the run as in Code-Burler-Review.
The writer can settle this by citing the landed commit, so it is no `decision` question for the parent.

### [LOW] A rejected alternative argued inside D6

**Location:** `_lym/discussion/decision-record.md:136`

**Issue:** The sentence ruling out the raddle store as the skip key argues an option not chosen.
The discussion spec keeps `## Decisions` to Decision and Rationale and puts rejected options in the support log.
The support log's Rejected alternatives covers the body-prefix key but not this one.
The sentence carries no requirement the plan writer needs: the chosen key and its clearing are stated in the sentences around it.

### [NIT] The no-Record-Write reason is stated twice

**Location:** `_lym/discussion/decision-record.md:43`

**Issue:** D1 and the last Constraints bullet both say that arras holds no Record-Write row because the loom contract allows one record row.

## Rejected

- Walkthrough 2, step 6 `EndEntry` would end the converted project done: D6 line 115 already says the filing write turns the entry into the project "instead of ending it done", which tells the plan step 6 does not run in the family outcome.
- D6 as a decision that edits the task's own board entry: the rubric's bullet rests on the run lock refusing the change. D6 changes board-module code to admit the change, which the bullet exempts, and the board entry itself asks for a project under the arras slug.
- Focus, D10: the rationale holds. `Store.Prune` removes done members and never closes a plain project, and `metBelow` counts a project as met only with at least one member, so a dependent left on the converted project would never start. The one-task outcome needs the re-point because a done entry counts as met at once. The record halts on a refused re-point and names the refusing entry, so no finding.
- Focus, D7 and Scope Out: the way forward after the filing names release of the part first, so it reaches a working recorder once the part's run passes its record row. The refusal is keyed on `perch` mode, which a `raddle` run never records, so it agrees with Scope Out. No finding.
- Focus, D8: the release lives in the verb's own path or a board method only the verb calls. The other null-status writers (batten cancel's claim clear, `fronttask` Release, beamcli, pairteardown) go through `Board.SetStatus`, so no finding.
- Focus, D4 in perch mode: the record says Raddle-Write ends on the finished proposal, the raddle gate's pass stores the cold proposal (`StoreCold` stores in any mode but interactive), and the proposal moves aside as in auto mode. This holds apart from the goto gap in F4.
