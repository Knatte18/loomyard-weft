```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-premised finding's search and check the fix covers every member.

**Findings**

1. LOW. `internal/dobbyengine/doc.go:421-422`, the doc of the beam-members landing notice (F1 and F3 site set).
   - The doc is stale against the fix.
   - Line 421 says each tick flags "every open member of the beam that lacks it". `flagMembers` now also skips a member whose listed run is past Land (`landnotice.go:203`).
   - Line 408 says the memory holds "whether the board entry carries `notify_on_land`". `LandNoticeState` now also holds `Member`.
   - Lines 426 and 429 do not say that a remembered member gets the follow-up line under per-task even when never flagged. That is F3's new behavior.
   - The code comments in `landnotice.go` were updated. This module doc is the one twin that was not, so a reader of it expects a re-flag and no unflagged-member line.

**Searches rerun**

- F1: every non-test writer of `SetNotifyOnLand` is `flagMembers` through `Deps.SetNotifyOnLand`. The seam closure is `dobbydeps.go:242`, the board method is `board.go:234`, and the rest are docs. No second flagging site was missed.
- F2: `PullRequestRecorded` has one closure, at `dobbydeps.go:238`. The new integration test covers it with all three cases, so the set is covered. `SetNotifyOnLand` was already covered by the existing seam test.
- F3 and F4: `isBeamMember` and `boardEntry` have only the `landnotice.go` call sites. `rememberRuns` has one caller, so no signature caller is broken. The only other `LandNoticeState{}` literal is `view_test.go:162`, and it is unaffected by the new `Member` field, which is `omitempty`.

**Tool calls**

1. `grep -rn "SetNotifyOnLand|NotifyOnLand: true|..."` for the F1, F2, F3 and F4 searches, because my context lacked the other sites. This also returned unrelated `boardEntry` helper hits from `fabricengine` tests, which I ignored.
2. `Read internal/dobbyengine/doc.go` lines 405-436, to check the module doc against the changed behavior. I had not read this file.
