---
round: 3
ledger:
  - key: serial-tests-no-named-reason
    rounds: [1]
    status: resolved
  - key: unsent-reprompt-cap-not-in-plan
    rounds: [1]
    status: resolved
  - key: send-go-header-narrower-than-file
    rounds: [1]
    status: resolved
  - key: leftover-strand-warn-on-every-reclaim
    rounds: [2]
    status: resolved
  - key: doc-restates-perishable-constants
    rounds: [2]
    status: resolved
  - key: stub-providers-no-idle-input-box
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
---

Round 1's three findings were resolved in round 2.
Round 2's two findings (`stopLeftoverStrand` logging, `doc.go` restating constants) were not raised in round 3 and are carried as resolved.
Round 3 raised one new BLOCKING finding, F1: the idle wait before every verified send leaves both loomcli stub providers, which print only `ReadyFooterFixture`, unable to read as idle, so their tmux-tier tests fail.
