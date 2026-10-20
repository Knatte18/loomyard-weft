---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:67
    summary: D3 says no goto grants a fresh budget after a budget-exhausted halt, but a budget_capped entry stops episodeStuckCount whatever follows it, so a goto after the halt grants one just as a resume does
    fix: In D3, state that episodeStuckCount stops at a budget_capped entry only when no OutcomeGoto entry lies after it; a goto entry after the capped entry makes the count continue past it, over the spent Stucks, so the row blocks again at its next counted Stuck and only a resume, which writes no history entry, grants the fresh budget. Say the same in D1's list of episode enders, where the budget-capped entry is named. Add an acceptance criterion that a goto onto the halted row, or onto a row of its segment, followed by a run leaves the count at the spent budget, while a resume of the same halt reads 0.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:128
    summary: D5 case 3 treats every settled verdict with no settle record as an older binary's, so a CONVERGED verdict the judge wrote after the driver died and before any harvest is cleared and re-reviewed, the loss D5 sets out to prevent
    fix: In D5, have the Bouncer write the generation record at every seed, generation 1 included, and amend "a directory without one holds generation 1" so that a directory without one is an older binary's. Then split case 3, or add a case 4. A settled verdict with no settle record in a directory that has a generation record is an unharvested verdict. The Bouncer settles it through its harvest path, which writes the record before carry-over, Approve and Commit. A directory with no generation record keeps today's clear and full re-seed. Add an acceptance criterion for the unharvested case.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:110
    summary: The told list of harness files the code-identity reader skips is an exemption with no stated bound; the record does not say what a change confined to those files lets through, what they may name, or whether tracked files are covered
    fix: In D5's code-segment bullet, add one sentence on the told list. It names exact worktree-relative paths of files the harness itself writes, such as `.scratch_task.json`, never a pattern or a directory. It exempts a path only while it is untracked, so a tracked file of that name still counts toward the identity. A change confined to listed files reads as an unchanged subject, bounded by the verify tiers D5 already names, which run over the whole tree.
---

### [MEDIUM] D3's "no goto does" is false under its own mechanism

**Location:** `_lyx/discussion/decision-record.md:67` (D3), with D1 at line 41 and the Open risk at line 215.
**Issue:** D3 marks the budget-exhausted Stuck `budget_capped`, makes `episodeStuckCount` stop at it, and then says that resuming grants the row one more budget "and no goto does".
The Goal repeats it: the halt is lifted "only by the parent's resume, never by a goto".
The count walks backward and stops at the capped entry, so it reads 0 plus whatever Stucks come after it.
`Goto` appends an `OutcomeGoto` entry and D1 makes `episodeStuckCount` skip it, so nothing a goto writes keeps the walk from reaching the capped entry and stopping there.
`gotoAdmitted` admits the run's current row and every earlier row.
A run blocked at the Bouncer therefore takes `goto --to <Bouncer>`, then the next step, and its first new Stuck reads a count of 0: the fresh budget a resume grants.
The Open risk at line 215 carries the same belief, since it says an older binary's halt gets no fresh budget from a goto under D1.
In the existing code, `Goto` appends `OutcomeGoto` and `episodeStuckCount` ends the episode at it.
So the premise is wrong for new halts too, and the criterion at line 225 pins only the resume's grant, never the goto's non-grant.
The harm is bounded, because the driver hands the halt back to the parent, so the tackler and the arbiter never goto it.
But the operator's decision that taking another round never resets the review budget stays open to a goto after the halt.

### [LOW] D5 case 3 reads a verdict the judge wrote but no Call harvested as an older binary's

**Location:** `_lyx/discussion/decision-record.md:128` (D5 case 3), with the generation record at line 103.
**Issue:** D5 case 3 reads "No record at all, and a settled verdict on disk" as a generation an older binary settled, and keeps today's clear and re-seed for it.
A new binary writes the settle record only when a Call harvests the verdict or accepts it, after the judge has written the verdict.
`Bouncer.Call` at `internal/shedadapters/bouncer.go:355` and following shows the other order: a judge that finished while no driver was attached is not attached to, and `settledGeneration` then clears the run directory.
A driver killed by a stop or a crash while the judge finishes alone leaves exactly that state: CONVERGED on disk, no record.
Case 3 would archive that verdict and re-seed a full review, which the Goal's "a stop keeps what the run already settled" and D5's rationale rule out.
A directory with no generation record cannot tell this state from an older binary's, because D5 writes a generation record only at an amendment or Merge-Review generation.
The window is narrow, so the severity is LOW.

### [LOW] The told skip list of the code identity has no stated bound

**Location:** `_lyx/discussion/decision-record.md:110` (D5, code segment).
**Issue:** The exclusion set is two things: the run's own state, bounded by `UncommittedPaths`'s own rule, and "a told list of files the reader skips, which loom and darn fill with the harness's own files, `.scratch_task.json` among them".
The attack-surface question asks the Decision to say what an exemption skips and what bounds it.
For the told list, the record names neither.
It does not say whether the list is exact paths or patterns, or whether it applies to a tracked file of that name.
It does not say what a change confined to those files lets through.
A pattern or a tracked match would let a task change escape both the identity and the amendment's reviewers, since the reviewers' current side reads the subject as the identity does.
The record's other exemptions each carry a bound, such as the verify bypass in D2 and the patch-id rule in D5.

## Focus verdicts

- Amendment seed over a spent episode: not a defect. D6 makes a `generation_start` Stuck skip Shed's budget comparison at the `!output.BudgetExempt && episodeStuckCount(...)` case of `stepLocked`, so a changed subject over a settle interrupted at the perch's last round or soft cap starts the amendment and is not blocked with `ReasonBounceBudgetExhausted`. The count then reads 1 as the entry is counted and the walk stops at it. The exemption is bounded by its once-per-generation seed, and the following Stucks of the generation are counted and compared. The criterion at line 236 pins it.
- Document segment base: not a defect. `CommitDiscussion` and `CommitPlan` stage the whole discussion or plan directory through `CommitAnchoredPaths` in the weft repo, so the weft HEAD read after Commit returns holds the settled artifact, including on a repeat commit that creates nothing. D6 states the interrupted-settle base, the weft HEAD at record write, and that an amendment then reviews the whole uncommitted artifact under the allowance.
- Code segment base: not a defect beyond F3. The stored patch the settle hashed and the current working-tree diff against the current merge base are read the same way, untracked files outside the exclusion set included. The run's own directories are excluded as `runOwnPath` excludes them, and a root `.scratch/` is already git-excluded by `seedScratchExclude`. Only the told list lacks a bound (F3).
- Merge-In recognition: not a defect. The test reads first-parent commits since the settle commit, so the parent's own commits on a merge's second-parent side are never in the range, and a fast-forward Merge-In is recognised by the commit itself being an ancestor of the recorded parent branch's tip. D5 states what the patch-id rule lets through and bounds it by the verify tiers. Darn's recipe has a Merge-In that verifies the merged tree and no Merge-Review perch, as D5 says.
