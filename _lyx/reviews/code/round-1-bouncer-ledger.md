---
round: 1
ledger:
  - key: removed-pattern-id-in-test-fixture
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: a parser fixture in `internal/discussionparser/validate_test.go` still names the removed ID `PATTERN-hub-suffix`.
The fix is a local replacement of the fixture ID.
No earlier ledger exists.
