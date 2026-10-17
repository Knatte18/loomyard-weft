---
round: 1
ledger:
  - key: card1-misses-creelcli-log-id-assertion
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: verify-omits-tools-sandbox-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card2-misquotes-unknown-receiver-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, none carried from earlier rounds.
F1 (BLOCKING): card 1 omits the `creelcli` subtest that D4 breaks.
F2 (MEDIUM, design): the `## verify:` section never runs `./tools/sandbox`, which card 1 edits.
F3 (NIT): card 2 quotes the wrong refusal text.
