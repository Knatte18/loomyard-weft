---
round: 1
ledger:
  - key: undeclared-pattern-overview-edits
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: resettle-swallows-circling-decision-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bare-pause-drops-earlier-by-label
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1, first round with no prior ledger.

- F1 (BLOCKING): `PATTERN.md` overview lines for `PATTERN-push-both-sides` and `PATTERN-leaf-packages` were rewritten though only `PATTERN-friction-capture` is declared; the fix restores the merge-base text.
- F2 (LOW): `resettle` in `internal/shedadapters/bouncer.go` ignores the `readCirclingDecision` error; the fix returns it wrapped and adds a test row.
- F3 (NIT): a repeat bare pause in `internal/shedverbs/pause.go` overwrites the recorded `--by` label with an empty string.
