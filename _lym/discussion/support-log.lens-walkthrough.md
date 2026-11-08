```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end and checked the end state against its purpose.

**Per decision: the case used and whether the purpose is met**
- **D1 (met).** Case: an arras task that is not yet discussed. The Motif block runs and the Discussion-Bouncer approve seam stores the record and sets discussed. The block exits to Raddle-Write, which reads the worktree record. `finishesAsSketch` is false because the recipe has a RaddleFile row. A cancelled and restarted run's `undiscussed` gate seeds the stored record and skips to Raddle-Write.
- **D2 (met, one LOW).** Case: the reviewer finds a bad seam. Bouncer → Burler-Review → Burler-Fix (overlay, raddle gate) → Bouncer → approved → Raddle-File.
- **D3 (met).** Case: an Added after Discussion entry that no part reflects. The reviewer reads the record through tool use and the fixer edits the proposal.
- **D4 (met).** Case: an arras run with `raddle_auto_approve` off. The mode is recorded as auto by rows, and Raddle-File's verify skips the approval-hash match.
- **D5 (broken for dependents).** Case: a one-task arras entry X that board entry Y lists in its `depends_on`. See finding 1.
- **D6 (met, with one step to name).** Case: a family filing. FileRaddle upserts the project under the run slug and files the parts under it. A retry finds `isRaddleFiledFrom` true through the body prefix. `SetRunStatus` from `battencli/commitstatus.go` is the write D6 must skip, and dobby starts only tasks (`dobbyengine/starts.go:62`), so it never starts the converted project.
- **D7 (holds before Raddle-File stages, not after).** Case: a decision added while Raddle-File is running. See finding 2.
- **D8 (met through the verb, ambiguous otherwise).** Case: releasing project P with members A, then B that depends on A. Dobby starts A, and B waits on `PendingDependencies`. See finding 3.
- **D9.** Prose only, nothing to walk.

**Findings**

1. **BLOCKING — D5 / D6: dependents of the arras entry are unblocked before the work exists.**
   - Summary: an arras entry is an ordinary task, so other entries may depend on it. In the one-task outcome Raddle-File marks it done through `EndRaddleEntry`, and the landing prune then removes it and drops it from every `depends_on`.
   - Effect: dobby's `PendingDependencies` (it counts only entries not done) empties, so dependent entry Y starts while the filed task is still a draft and nothing is built.
   - Why raddle never hits this: a Sketch with dependents is refused by `MakeSketch`/`validateWrite`, so a raddle run's entry has none. An arras task has no such guard.
   - The family outcome is unaffected only by luck: dependents then wait on a project, and the record does not say when or whether a project counts as done.
   - Fix: the filing write re-points dependents' `depends_on` from the arras slug to the filed task (or keeps them on the project), or the arras start refuses an entry that has dependents. State the bound either way.

2. **MEDIUM — D7 / Open risks: a decision added once Raddle-File has staged is lost from every part, and the Open risk misstates it.**
   - How Raddle-File builds the records: step 1 stages the record (`stageRecordFile`), and step 2 writes each part's record from that staged copy (`raddlefile.go` `filing` reads `stageRecordFile` and `PartRecordPath`). In the family case, FileRaddle then writes the staged whole record as the project's `decision-<slug>.md`.
   - Effect: that write overwrites the copy `copyRecordToBoard` just stored.
   - Errors in the record:
     - The Open risk says a late decision "reaches every part's record". That holds only up to staging.
     - D7's "every filed part's record carries every Added after Discussion entry" holds only before Raddle-File.
   - After Raddle-File, `pastApproval` is still true, so `decision add` succeeds silently and stores only onto the project, or onto an entry the prune is about to remove.
   - Fix: bound D7. `lym loom decision add` on an arras run at or after Raddle-File should refuse by rows and name a way forward, such as `lym loom goto --to Raddle-Write`. Then correct the Open risk.

3. **MEDIUM — D8 / D6: the release trigger is unbounded across write paths, and D6's filing write leaves the project's status null.**
   - Release paths: D8 names only `lym board set-status`. The board also releases through an upsert with `status` null and a merge whose `set_status` is null (`boardengine/doc.go` Draft section).
   - Effect: if the plan puts the release rule at the write level ("a write leaving a project's status null"), D6's conversion, which clears the entry's run status to null in the filing write, would release every part as it is filed. That defeats D4's "every filed task stays draft until the operator releases it".
   - Fix: D8 should state the trigger exactly, as the set-status verb, or as a project's status written to null by a write other than the raddle filing, and add a criterion that a family filing leaves every member draft.

4. **LOW — D2 vs Rejected alternatives: inconsistent on new seams.**
   - D2's "the ring commits a fixed proposal the way Raddle-Write commits" implies a new Bouncer `commit_seam` value. That is a closed set of Env closures, the same kind of addition the support log rejects for approval.
   - Raddle-File's step 2 already commits the raddle directory, so the sentence can say the fixed proposal is committed by Raddle-File's commit, or name the seam outright.

5. **LOW — D1 / D4: re-entering the perch after a Raddle-File return.**
   - After a return, Raddle-Write moves the proposal to ReturnedPath and its on_done re-enters Raddle-Bouncer, which already approved once in this segment.
   - The record does not say whether the segment's bounce budget restarts. A second pass may open at hard-cap and go straight to the arbiter.
   - This belongs in Notes for the plan writer beside the existing segment note.

**Tool calls**
1. `grep` for `boardRecipe` in `battencli/wire.go` and for entry/end handling in `loomshed/raddlefile.go` and `loomcli/raddlewiring.go`: needed to see how a project slug resolves (to pawl) and how Raddle-File reads and ends its entry, which the context lacked.
2. `sed` of `raddlewiring.go` 155–225 and `raddlefile.go` 110–175 and 378–420: needed Raddle-File's step order, the staged record source and the retry path, to walk D6 and D7.
3. `grep` for batten end writes and dobby project handling: needed to know whether batten marks the entry done and whether dobby starts a project entry.
4. `sed` of `battencli/commitstatus.go` 110–150: needed the batten run-status writer that D6 must skip after the conversion.
