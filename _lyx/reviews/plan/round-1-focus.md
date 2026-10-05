---
round: 1
focus:
  - "Card 16 Intent: does the shared cwd-resolution helper extracted from `runConfig` keep bare `lyx config` and the unknown-argument refusal resolving no cwd (decision D9, constraint `PATTERN-cwd-resolution`), or does the extraction pull cwd resolution ahead of them?"
  - "Card 16 and the attack-surface question: the new `menu` routing edge states its bound in Intent; settle whether the bound covers the interaction with `Args: cobra.NoArgs` and the D9 rule that an argument naming a subcommand always routes as one."
  - "Card 15 Intent: the editor resolution order against `_lyx/loom/rework/round-1/findings.md` (`$VISUAL`, `$EDITOR`, `code --wait`, `nano`, `vi`, notepad on Windows); settle whether any card step adds scope that finding does not license, and whether the test list covers each branch of the order."
  - "Card 15 and card 16 both edit `internal/configcli/configcli_test.go`, `doc.go` and `buildConfigLong`: settle whether the split into two cards is independently reviewable, or whether the shared files make the sequencing or batching a `design` problem."
  - "`## verify:` against the cards' targets: `internal/configengine`, `internal/configcli`, `docs/overview.md`, and the `cmd/lyx` tree walk; settle whether every package a card targets has its tests run, including integration-tagged ones."
  - "Card 16's `Uses` and `Edit` targets: settle whether the `Uses` entries (`cmd/lyx#TestCLITree_EveryCommand`, `internal/configcli#editOne`) are real dependencies the card reads, and whether each card's `ImpactSummary` states a blast-radius conclusion rather than restating Intent."
---

Seed focus for a two-card rework plan (editor fallback, `lyx config menu`).
Mechanical format checks and derived dependencies are left to the gate, per the rubric's `Do not flag` list.
