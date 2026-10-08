---
round: 4
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
    status: resolved
---

Round 1's three findings were resolved in round 2, and round 2's two findings were not raised again.
Round 3's BLOCKING finding, the loomcli stub providers lacking an idle input box, was fixed by commit `00b9be707`.
Round 4 found nothing, so every key is resolved.
