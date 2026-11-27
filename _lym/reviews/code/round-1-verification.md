---
round: 1
entries:
  - key: tmuxrec-test-file-names-two-tiers
    status: closed
    finding: F1
    reason: "245270036 moves the tier2 test and its tmuxkit, exec and proc.KillPID uses to recordings_realtmux_test.go and the scenarios to scenarios_test.go; the tier-tag and tier-purity guards pass"
  - key: tmuxrec-kit-asserts-beyond-setup
    status: closed
    finding: F2
    reason: "245270036 removes the t.Errorf cleanup from PlaySession; TestRecordings_Replay and newPRSessionWorld now fail on replay.Unanswered(), naming the recording and the calls"
---
