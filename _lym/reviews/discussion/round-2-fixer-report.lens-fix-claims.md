```yaml
lens: fix-claims
outcome: reported
```

Fork task: check that every claim the fix added or rewrote holds against the code at HEAD.

**1. BLOCKING. D4's new bullet, and D5's new crash paragraph, rely on an abort that does not behave as claimed.**

Changed text:
- D4: "concluding the merge then carries them, and aborting it keeps the records commit"
- D4's way forward: "with `lym fabric merge --abort` for one fabric started"
- D5: "The next run of the merge verb aborts the merge and starts again from step 1. `mergeresolve` aborts an in-progress merge before its attempt the same way today."

Summary: fabric's own abort does not run `git merge --abort`. Today, after D4's in-merge records commit has moved HEAD, it either refuses or would hard-reset that commit away. The named way forward dead-ends, and so does D5's crash resume.

Evidence:
- `internal/fabricengine/mergelifecycle.go:334-372`: `MergeAbort` resets the warp side with `resetMergeSides(rec, st.WarpStart)`, a hard reset to the recorded pre-merge SHA (`internal/fabricengine/destroy.go:2492-2501`, `resetHardTo`). It does not reset to the current HEAD.
- Before that reset, `concludeLandedReason` (`internal/fabricengine/mergeguards.go:257-272`) calls `sideConcludeMayHaveLanded` (`mergeguards.go:373-386`). For a staged or conflicted outcome that returns `head != start`. A records commit that moved HEAD mid-merge therefore reads as a conclude that may have landed, and the abort refuses with `mergeReasonConcludeLanded`.
- `mergeresolve.Resolver.Resolve` (`internal/mergeresolve/mergeresolve.go`, crash-recovery block) aborts through `r.deps.Fabric.MergeAbort()`. After a crash in which a records commit landed between step 3 and step 4, the resume hits the same refusal instead of "starting again from step 1".
- The Constraints pin about `git merge --abort` being `git reset --merge` holds for plain git only. Fabric's verbs do not use it.

The record states none of the changes these claims need: the conclude-landed guard reading HEAD past `WarpStart`, and abort resetting to `WarpStart`.

**2. LOW. D5's crash sentence overstates what is in the index.**

Changed text: "one before step 3 leaves the source's records in the index".

Summary: this holds only for records paths the source changed. A source that changed no records path leaves the index equal to HEAD, so D4 lets records commits through rather than holding them. The sentence states the hold as unconditional.

Evidence: D4's own condition, "It does so only while the index holds every records path as HEAD does".

Every other claim I checked holds against HEAD:
- The partial-commit refusal is quoted in `failureDisposition`'s doc (`internal/statuscommit/statuscommit.go:146-168`).
- `mergeresolve` does abort an in-progress merge before its attempt.
- `lym fabric merge --abort` exists (`internal/websterengine/gitwrap.go:41-44`).
- `IsPushOwed` and `FailedPushesOwedBy` read only the transport, deferred and teardown_lock classes today, which matches the stated change.
- The support log's count of five blocking findings and one medium matches the review.

Tool calls:
- Grep for `MergeAbort` and `MergeContinue` in fabricengine: my context had no fabric merge-lifecycle code.
- `mergelifecycle.go:225-400`: to see what `MergeAbort` and `MergeContinue` reset to and check.
- Grep for `resetMergeSides`, `concludeLandedReason` and `recordedMergeGoneReason`: to locate the abort's reset target and its guard.
- `mergeguards.go:200-272` and `sideConcludeMayHaveLanded`: to confirm the guard compares HEAD with the recorded start SHA.
