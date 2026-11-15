```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1 and F5; F2, F3 and F4 get one-line notes).

**Finding 1 — MEDIUM — F1, the branch discard in `discardTaskBranchIfPresent`, `clearDoneRun` and the `ClearDoneRun` doc ("A failed branch removal leaves the run's records in place")**
- **Summary:** The origin half of `DiscardTaskBranch` can fail after it returns nil, so the records are removed and the slug is freed with the origin branch still present.
- **Evidence:** The `DiscardTaskBranch` doc in `taskbranchdiscard.go:30-37` says it deletes the local branch, "then hands the remote half to the holder of the slug's teardown lock, and returns without waiting on it". It also says a failure in the remote half "is logged and mailed to the hub's orch", not returned. The code at lines 91-97 returns `res, out.err` only when `out.ran`, meaning the remote half ran in process.
- **Case:** A done run's task branch is on origin and another process holds the slug's teardown lock, so the remote half is handed off. `DiscardTaskBranch` returns nil, `clearDoneRun` removes the records, and the slug is reported cleared. The origin deletion then fails, for example on a lost lease. The run directory is gone, so the next tick has nothing to retry. A later `lym beam merge-parent` or start under the slug is refused by the leftover origin branch.
- **Why it matters:** This is the failure F1 set out to close. The doc claim that a failed branch removal leaves the records holds only for the local half and the in-process remote half.
- **Possible change:** After a nil discard, confirm `TaskBranchOnOrigin` is false before removing records, and leave the run for the next tick otherwise. Alternatively, narrow the doc claim to say so.

**Other notes, no finding:**
- F5: the added `wantMessage` assertions do not conflict with any sibling.
- F4: the proposed drop of counted entries for a slug that is neither landed nor listed can only re-count a landing if `Landings()` omits a landed run for one tick. That is speculative and below LOW.
- F2: the proposed doc text matches the code.
- F3: the test pins the `d.landed` link but not the tick call placement. The fixer chose the `countLandings` then `clearDoneRuns` option the review allowed.

**Further calls:** none. My context already held every file I needed.
