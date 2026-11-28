---
round: 1
entries:
  - key: printed-wait-includes-in-slot-work
    status: closed
    finding: F1
    reason: "d6c837e86: runWithSlots stamps wait.held once slots are held or unneeded, before the shared build and copies prebuild; SlotWaitSeconds ends at SlotsHeld (slots_held), falling back to Start; summary.md rewritten; slotwait_test.go has the shared-build row."
---
