---
round: 1
entries:
  - key: repeated-literals-not-named-constants
    status: closed
    finding: F1
    reason: "c3bd1d6bb: probeFolderPrefix is passed at all three MkdirTemp sites and builds the test glob, report.go prints trapsPath through %s, and lymStartName feeds both the measureStarts call and the Skipped text"
---
