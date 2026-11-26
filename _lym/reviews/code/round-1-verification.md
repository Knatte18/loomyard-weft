---
round: 1
entries:
  - key: cancel-rerun-refuses-with-only-origin-records-branch
    status: closed
    finding: F1
    reason: "a5bb4acd5: refuseNothingLeft in internal/battencli/cancel.go counts a pending teardown record naming this cancel as left under --remove, and TestCancelIntegration_FinishesAnUnfinishedRemoval drives the re-run from the origin-records-branch-only state and asserts the branch is deleted"
  - key: fabric-docgo-describes-batten-cancel
    status: closed
    finding: F2
    reason: "a5bb4acd5: the batten cancel sentence is deleted from internal/fabricengine/doc.go, and the sentence added in its place states only fabricengine's SettleGonePairTeardown behaviour"
---
