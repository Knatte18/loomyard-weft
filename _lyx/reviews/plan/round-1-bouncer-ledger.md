---
round: 1
ledger:
  - key: card7-fresh-parse-refusal-unbacked
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card7-kickstart-pack-rows-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-binderror-row-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card6-renderfork-resign-arrow-and-caller
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card3-ownedhandles-bundled
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card6-d8-line-in-go
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: card6-pack-fragment-spec-pointer
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised seven findings, all open.
The blocker is card 7's new refusal on a failed fresh plan parse, which has no refusal-spec row, reaching input or test.
Three further design findings concern unreachable test rows in cards 4 and 7 and the bundled `OwnedHandles` move in card 3.
Card 6 carries one scope finding (re-sign arrow and unlisted caller) and two low decision findings.
