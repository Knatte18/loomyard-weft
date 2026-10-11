---
round: 2
ledger:
  - key: card7-fresh-parse-refusal-unbacked
    rounds: [1]
    status: resolved
  - key: card7-kickstart-pack-rows-unreachable
    rounds: [1]
    status: resolved
  - key: card4-binderror-row-unreachable
    rounds: [1]
    status: resolved
  - key: card6-renderfork-resign-arrow-and-caller
    rounds: [1]
    status: resolved
  - key: card3-ownedhandles-bundled
    rounds: [1]
    status: resolved
  - key: card6-d8-line-in-go
    rounds: [1]
    status: resolved
  - key: card6-pack-fragment-spec-pointer
    rounds: [1]
    status: resolved
  - key: card8-parse-degrade-row-refused-upstream
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card8-kickstart-test-missing-impactsummary
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card5-bindtrace-fixture-lacks-plan-files
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings; none reappears in round 2, so all are resolved.
Round 2 raised three new findings on cards 8 and 5, all about test rows whose fixtures cannot reach their branch as written.
The one gating finding, the parse-error degrade row in card 8, is on a key first raised this round.
