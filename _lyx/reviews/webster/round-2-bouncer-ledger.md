---
round: 2
ledger:
  - key: shuttle-stop-entry-wording-departs-from-card
    rounds: [1]
    status: resolved
---

Round 1 raised one finding: the `PATTERN-shuttle-stop` line in `PATTERN.md` was reworded in both its lead-in and its exception clause, where the card asked for only the clause after the semicolon.
Round 2 found no finding on it.
The reviewer showed that the card's verbatim text makes the entry line exceed `MaxEntryLineChars = 320` in `internal/pattern/check.go`, which `TestOverview_RealFilePassesCheck` enforces, so the shortened wording is required and the departure is not a defect in the diff.
