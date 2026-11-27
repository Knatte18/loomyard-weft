---
round: 1
entries:
  - key: refused-record-survives-owned-elsewhere-settle
    status: closed
    finding: F1
    reason: "46dd55a61: settleOnOrigin's owner branch removes the landing, owes the hand-over and calls clearUnlessDropped, writing no new record; row ARefusalOverARunOwnedElsewhereEndsOnceItsTipIsOnOrigin covers it"
  - key: failed-reset-or-handover-lost
    status: closed
    finding: F2
    reason: "46dd55a61, 91fa82cb2: the entry keeps Dropped and HandOvers, payOwedSteps retries both in every pass and SettleRefused and records a failure as held_back, with no landing left too; rows cover a refused reset and a failed hand-over"
  - key: beam-parent-pusher-never-restarted
    status: closed
    finding: F3
    reason: "46dd55a61, 5cf633ae6: restartHubPushers calls RestartPusher with start(target), keyed by LockPath, for each parent target that does not read refused; TestRestartHubPushers asserts the beam target is started"
  - key: test-only-fabric-methods
    status: closed
    finding: F4
    reason: "e7ae7c945: LocalOnlyCommits and its test are deleted, PushBranch and PushWarpRebaseFreeAt moved to export_test.go as PATTERN-test-only-code allows, and doc.go no longer lists PushBranch"
---
