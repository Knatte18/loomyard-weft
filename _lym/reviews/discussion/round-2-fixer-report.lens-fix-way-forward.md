```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each way forward, refusal and row the fix added can be carried out and makes progress.

Ways forward the fix introduced:
- D4: the transition waits while the merge is unfinished. Way forward: "conclude or abort the merge, with `lym fabric merge --abort` for one fabric started".
- D5: a crash between step 2 and step 4 is resumed by "the next run of the merge verb", which aborts the merge.
- D7: when the `lym.primeBranch` key is absent, the records commit refuses. Way forward: run `git config lym.primeBranch <branch>`, then run the step again.
- D7 bound: an `operator_unpushed` merge is owed until a landing's push of its parent or the operator's `git push` carries it.
- D7 protected: the run continues with its records unpushed.
- Acceptance rows added under D4, D7, D8 and D9.

Findings:

1. **BLOCKING** — D4: "aborting it keeps the records commit" and the way forward "with `lym fabric merge --abort` for one fabric started". Also D5's resume: "The next run of the merge verb aborts the merge".
   - Summary: fabric's own abort does not run `git merge --abort`. It either refuses to abort, or it resets hard to the pre-merge start and throws away every records commit D4 made during the merge, which may already be pushed.
   - Evidence: `Fabric.MergeAbort` (`internal/fabricengine/mergelifecycle.go:334-378`) first runs `concludeLandedReason`, which asks whether HEAD moved off `st.WarpStart` (`internal/fabricengine/mergeguards.go:257-271`). A records commit made during the merge moves HEAD, so the abort can refuse with `mergeReasonConcludeLanded`.
   - If the abort does go ahead, `resetMergeSides(rec, st.WarpStart)` runs `resetHardTo` to the recorded start SHA (`internal/fabricengine/destroy.go:2492-2501`). That drops the transition commits made during the merge.
   - `mergeresolve`'s resume calls the same `MergeAbort` (`internal/mergeresolve/mergeresolve.go:41-49`).
   - The Constraints tool fact the fix added ("`git merge --abort` resets to HEAD") is true of plain git, but not of the path the record names.
   - So the named way forward either cannot be carried out, or it undoes the transition it was meant to unblock. D4 needs to say that fabric's abort and continue tolerate, and keep, a records commit moved onto HEAD during the merge.

2. **MEDIUM** — D4: "conclude or abort the merge", read together with D5's "one before step 3 leaves the source's records in the index, so D4 holds records commits until the merge is aborted".
   - Summary: two problems with the crash state.
     - Concluding is named as a way forward, but concluding a merge that crashed before step 3 commits the source's records into the receiver. That breaks D5's rule.
     - The resume relies on "the next run of the merge verb", but the run is waiting at its transition and will not start the producer that runs that verb.
   - The only signal is a log warning, so in practice nothing moves until an operator acts.
   - Evidence: D4's wait rule ("the transition seam ... waits in place of its skip ... with a warning"). The seam's dispositions (`internal/statuscommit/statuscommit.go:58-78`) send no creel notice for a wait. D5 step 3 is the only step that restores the receiving side's records.

Further tool calls:
- grep of `internal/fabriccli` and `internal/fabricengine` for `--abort` and `MergeAbort`: my context lacked whether the named verb exists and what it does.
- `sed` of `internal/fabricengine/mergelifecycle.go:346-420`: the body of `MergeAbort`, which my context did not hold.
- grep for `resetMergeSides` and the merge-state file: what the abort resets to, which my context did not hold.
- grep for `concludeLandedReason`: the guard the abort runs first, which my context did not hold.
