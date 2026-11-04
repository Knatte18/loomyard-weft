---
verdict: CONTINUE
rationale: "Round 1 carries four design findings at MEDIUM (F1, F2, F3, F4), which are gating; no finding was relabeled and there is no Focus departures section."
---

F1 is a behavior gap: `answerReopen` in `internal/landingshed/prreview.go` leaves a merged pull request's record in place.
F2, F3 and F4 are coverage gaps on the reopen path: `RemoveReady` and the `Unready` closure, `readReopenRequest`, and the pr-reviewer stop.
F5 and F6 are LOW design findings, and F7 and F8 are LOW consistency findings; none gates.
The rejected items in the review stand as written.
