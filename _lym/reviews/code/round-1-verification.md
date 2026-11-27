---
round: 1
entries:
  - key: beam-with-idle-members-still-refused
    status: closed
    finding: F1
    reason: "3f67014ea and 915fe99e1: refuseBeamCancel takes remove and workingBlockers drops idle task members in the held and unheld branches, with table rows and summary.md updated; idle sub-beams still refuse on purpose, as one may be open on another machine."
  - key: endbeam-fake-stand-in-no-real-board-test
    status: closed
    finding: F2
    reason: "3f67014ea: the fake EndBeam only records the call and returns false, the made-up board error row is gone, and TestCancelIntegration_RemoveOfABeamLeavesAPlainProject checks the cleared fields, beam_ended and the beam=false upsert over a real hub board."
---
