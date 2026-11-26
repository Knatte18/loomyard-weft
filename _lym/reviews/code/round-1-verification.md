---
round: 1
entries:
  - key: moved-checks-way-forward-cannot-clear-halt
    status: open
    finding: F1
    reason: "9041bf93a/685e3268f add the moved-check rule, but rangeRulingKey (loosened.go) hashes only CommittedFileDiffs (--diff-filter=MD, taskpatch.go:91), so moved checks committed in a new test file leave it unchanged and resume halts with no judge; a carried_by ruling naming a new file is likewise reused after that file drops the check."
---
