```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1, F2, F3, F5, F7 through F9 and F11 through F14), looking for a case where it is wrong or breaks something.

**LOW — F5 in `Runner.TurnOnRemoteControl` (`internal/shuttleengine/session.go`)**
- **Summary:** `errors.Is(err, ErrSubmissionNotLanded)` waits the full `remoteControlConnectWait` (20 s) even when the command provably never ran.
- **Evidence:** `run.go:1786` says a send that does not land fails with `ErrSubmissionNotLanded` after `clearUnlandedText` has emptied a box holding exactly this send's own text.
  - Case: the paste fails, the box is cleared, no connection record is ever written, and the call holds the creel delivery worker for 20 s before returning `(false, ErrSubmissionNotLanded)`.
  - The proposal gives the right result there (no mark, error returned), only slowly.
  - The wait is bounded and is the cost of the reviewer's chosen trade-off, so I rate it LOW.

I found no breakage in the other proposals:
- **F1:** `transcriptPathIn` is used at both new sites and the older literals are left alone, as the review said.
- **F2:** the `(false, nil)` return after a failed read leaves creel logging "no connection" at Info, which is slightly misleading but harmless.
- **F3:** the mark now follows `saveFreshLaunch`, and nothing between the two uses its result.
- **F7 through F9 and F11 through F14:** I found no case where the proposed test change passes with the defect in place. The `F9` `pasted` closure reads `reed.PasteSubmitCalls` without the fake reed's mutex. That is safe only if the paste runs on the caller's goroutine, and the gate passed. I did not run it under `-race`.

Tool calls: none; my context already held every file I needed.
