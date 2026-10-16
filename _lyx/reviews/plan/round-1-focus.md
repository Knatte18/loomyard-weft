---
round: 1
focus:
  - "Card 1 targets `internal/websterengine#TestCardGateCommand` and `internal/batcher/estimate_test.go` while the Shared Decisions bar member glyphs on external `_test` packages: settle whether each target's package is internal or external, and whether each Edit card's Intent names the symbols its targets list, with the re-sign arrow wherever a signature changes."
  - "Cards 2 and 3 each add a refusal (`verify-package-missing`, the `lyx gate test` package-directory refusal): settle for each that the Intent names an input that reaches the branch and a test that exercises it, that no upstream check rejects that input first, and that the bound on what it can let through is stated."
  - "Card 4 changes the loop's stop causes (`status-missing`, `status-unreadable`), the re-read, the trace file and a stated departure from `PATTERN-failure-stop-visible`: settle that every decision-record D5 bullet and the two refusal-spec rows are carried by a card, that a reachable test input exists for each branch (absent file, decode error, read error clearing on re-read, read error failing it), and that the card states what the unwritten status lets through and what bounds it."
  - "Fidelity: settle that every Decision D1 to D7 and every Constraint (docs in the same commit, `cmd/lyx/testdata/test-budget.yaml` under `Edit:` where a package passes its row, `PATTERN-producer-pointer-rule` for card 5) is carried by some card, and that no card adds scope the decision record does not license, such as card 5's D6 note or the `lens-review.md` file in the plan directory."
  - "Card order and `## verify:`: settle that no card depends on a later card's output (cards 2 and 3 on card 1's mapping, card 4 on its own packages), and that the verify chain runs the tests of every package any card targets, including `internal/shedverbs`, `internal/gatecli`, `internal/planparser`, `internal/websterengine`, `internal/batcher` and `cmd/lyx`."
  - "Test economy: settle whether each new test function duplicates behavior an existing table or test already covers (the decision record asks to extend existing tables), naming that test."
---

Seed focus drawn from the overview, the five cards and the decision record.
No instance is promoted without checking it against `Do not flag`: the dependency-edge and `ImpactSummary` omissions are excluded.
