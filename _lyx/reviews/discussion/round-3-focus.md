---
round: 3
focus:
  - "In the decision record's D5, the failure-record freshness window compares the record's `written_at` with the child's shed history `At` values, which `nowRFC3339` in `internal/shedengine/run.go` stamps at whole-second precision. Settle whether D5 states the precision of `written_at`, which bound of the window is inclusive, and what happens for a genuine record and a stale same-HEAD record written in the same second as their halting entries."
  - "In D5, the goto leaves the child paused at its target until the resume follows, and `noticeCondition` in `internal/battenshed/notice.go` returns a state-changed notice for every non-running state. Settle whether D5 states which notices are suppressed while the retry is pending, what notice batten sends past the retry's time cap, and whether the paused arm resumes on every Call or only after a restart."
  - "In D2, the deploy command runs task-authored code on the strength of PR-Gate approval of the landed tip, but `PRGate.Call` in `internal/landingshed/prgate.go` returns Done without a review when the parent branch is outside landing config's `require_pr_to_base`. Settle whether D2's stated bound and D1's prime check hold for a prime branch outside that list, and whether the decision names how that case is handled."
  - "In D6, a remote cannot-lock-ref refusal today maps to `ErrPushRejected`, which `pushRebaseFreeLogged` in `internal/fabricengine/coalesce.go` turns into a warning with the commits left unpushed. Settle whether D6 states what each caller class receives when retries run out (Publish's Stuck, the coalesced push path) and carries that behaviour into its acceptance criterion."
  - "The Scope section says the push retry covers every fabric push, while D6 names the covered primitives and excludes the leased update and the remote-branch deletes. Settle whether the two agree."
---

The entries restate each site they depend on and ask only for a check of the decision record against the code it cites.
