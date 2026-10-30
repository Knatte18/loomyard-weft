```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F6), as an outsider.

**1. MEDIUM — F1: the merged-at-ungated-head halt is still bypassed on any wake where no poll is due.**
- Changed text: `wake` in `internal/landingshed/prreview.go`, where `merged := polled && status.State == pullrequest.StateMerged` guards the gate.
- Case that still fails:
  - The record has `GatedHead` h1, and the task head is h2, ungated.
  - Call's entry `PushTask` pushes h2, and the entry status is open.
  - The first wake uses the entry status as its poll, and nothing is merged yet.
  - The pull request is merged at h2 on GitHub during the next minute.
  - A later wake has `polled == false`, because `pollStatus` returns early when `now.Sub(lastPoll) < prPollInterval`.
  - So `merged` is false, and `gateHead` gates h2, pushes it, and sets `GatedHead = h2`.
  - The next due poll sees merged at h2 equal to `GatedHead`, and `judge` returns Done instead of the merged-with-other-head halt.
- Why it matters: the proposal's premise holds only when the merge is already visible in the same wake.
  The new row `MergedAtAnUngatedTaskHeadHaltsWithoutGatingOrPushingIt` passes only because the entry status (merged) is consumed as the first wake's poll.
- A sound fix would be to compare `status.HeadSHA` with the `GatedHead` the record held before this wake's gate. Another is to force a status read before gating an ungated head.

**2. LOW — F2: a failed stop is only logged, and the record is then removed anyway.**
- Changed text: the new `p.stopLiveSession(rec)` Warn block before `RemoveRecord` in `closeRecordedPullRequest`.
- Case: `StopStrand` or `SessionLive` fails, and the record holding the strand GUID is dropped regardless.
  The session is then unrecoverable, because `endSession` finds no record and no later call can find the strand.
  Returning the error would keep the record, so a re-step could retry the stop.
- The reviewer's "log a failed stop" wording matches `endSession`, but `endSession` runs when the record is kept, so its situation differs.

**3. LOW — F4: an unreadable fabric config now suppresses the gone-worktree Warn.**
- Changed text: the new `fabricengine.LoadConfig(...)` branch in `closePairPullRequest`'s `gone` path.
- Case: the config load fails in the gone-worktree case.
  The function warns about the config and returns without logging "may stay open", which the old code always logged.
- The gone path did no config read before, so the change adds a new failure mode.
  Falling back to `slug` as the branch attribute would keep the warning.
- The new `closePairPullRequest` gone-path code is also untested. `TestCloseOpenPullRequest` covers `closeOpenPullRequest` only.

No finding for F3, F5 or F6.
- F3: `StopStrand` on the failed record write holds up.
- F5: the doc now matches the code.
- F6: the new row fails if the `!gated` half of the guard is dropped, because `Close` would then appear in `wantCalls`.

Further tool calls: none. My context already held every file I needed.
