---
round: 2
entries:
  - key: range-key-omits-added-files
    status: closed
    finding: F8
    reason: "12b1c5e16 (comments rewrapped in d4c31d167): listFiles hashes the new RangeDiff seam, wired to ReadCommittedDiff, so added files count; seam required, comments and doc.go updated, both rows in verify_test.go use an added file"
---
