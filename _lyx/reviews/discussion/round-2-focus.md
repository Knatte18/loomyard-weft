---
round: 2
focus:
  - "D5 failure-record freshness: in `internal/landingshed/verifygate.go`, `checkVerify` and `checkPublishVerify` call `recordFailure` (which writes the verifytree publish-failure record) before returning the Stuck reason, and shed sets the child's blocked state only after the producer returns. Settle whether the freshness test D5 states can ever accept a genuine record, and whether it rejects a record left by an earlier halt at the same HEAD."
  - "D5 retry loop: D5 suppresses the blocked notice while it retries a goto refused under the run lock or a resume refused as not parked. Settle what bounds the retries within one episode, and what the orch sees if the driver never releases the lock or never parks. `noticeDriverBusy` in `internal/battenshed/notice.go` is the existing driver-busy notice."
  - "D2 trust bound: the configured deploy command runs `./update-plugins.sh`, which execs `go run ./tools/deploy` from the landed tip and pushes `prod` outward; `hubWriteRefusal` in `internal/configcli/hubwide.go` documents itself as a guard against a mistaken write only. Settle whether D2 names the true trust anchor and what the command can do, not only when it runs."
  - "D6 push retry: `rebaseRetryTriggers` in `internal/gitrepo/push.go` matches `rejected`, so a remote `cannot lock ref` refusal already triggers `pull --rebase` in `Push` and `PushCoalesced` and maps to `ErrPushRejected` in `PushRebaseFree`. Settle which push primitives D6's retry covers (including the leased force push and the delete forms), where it sits relative to that classification, and what surfaces when attempts run out."
  - "D4 failure episodes: the Land-failure notice names `lyx batten run <slug>` as the way forward, which re-runs the row from the top, and a marker is meant to stop a batten restart repeating the notice. Settle what ends a Land or teardown failure episode, so a deliberate re-run that fails again is distinguished from a crash-restart mid-notice."
---

Round 2 focus targets the Decisions whose bounds and orderings depend on code behaviour that the discussion must state correctly.
