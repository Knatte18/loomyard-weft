---
round: 1
ledger:
  - key: d3-legacy-list-under-open-map
    rounds: [1]
    status: open
  - key: d3-open-map-lets-through-unstated
    rounds: [1]
    status: open
  - key: d7-validation-order-cross-entry-duplicates
    rounds: [1]
    status: open
  - key: d9-precedence-and-flag-combinations
    rounds: [1]
    status: open
  - key: scope-in-omits-picker-removal-and-allowlist
    rounds: [1]
    status: open
  - key: r3-repeats-scope-out
    rounds: [1]
    status: open
---

Round 1 raised six findings, none gating.
F1 and F2 (MEDIUM, decision) concern D3's open-map behavior: `--set` under a legacy list, and what `MissingKeys` and a mistyped label name now let through.
F3 and F4 (LOW, decision) leave D7's validation order and D9's precedence open.
F5 (LOW, consistency) asks Scope In to name the picker removal and the allowlist deletion.
F6 (NIT, consistency) asks to trim R3.
All stay open until a fixer addresses them.
