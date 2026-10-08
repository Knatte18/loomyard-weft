---
round: 1
focus:
  - "Card 4 and the decision record's `PATTERN-spawn-observability` constraint: the constraint says a timed-out git kill is logged through `internal/logger`, while card 4 makes the kill's `report` callback do nothing and relies on callers logging the `*GitError`. Settle whether some card carries that constraint or the plan's stated exemption leaves it uncarried."
  - "Card 4 test economy and verify coverage: new `transport_test.go` rows and the new integration test against the existing `transport_integration_test.go` and `transport_test.go` tests. Settle whether any planned row duplicates behavior an existing test covers."
  - "Card 2 test economy: the new `parity_test.go` table, the `gogit_test.go` subtests and `gogitcaller_test.go` against existing tests (`TestSHAExists_MixedBackend_RepackBetweenCommitAndRead`, the `concurrent callers` subtest). Settle whether a new function or file covers behavior an existing test already does."
  - "Edit-card member glyphs and the re-sign arrow across cards 2, 4 and 6: for each member whose `Intent` changes parameters, results or receiver (`runCore`, `RunGit`, `Run`, `GitError`, `Repo.goGit`, `checkRunStatus` extraction from `battenPreRun`), settle whether the glyph needs the arrow. Interface methods and struct fields take none."
  - "Card order: card 1 edits `HeadContains` and card 2 moves its walk into `readGoGit`; card 4 `Uses` `proc.ConfigureGroupKill` from card 3; card 5 relies on room card 2 frees in `PATTERN.md`. Settle whether any card depends on something a later card does."
  - "Attack-surface statements in `**Intent:**`: cards 4 (deadline, kill, transport classification of timeouts), 5 (auto-gc and maintenance switched off, gc at removal) and 6 (new refusal path). Settle whether each states what it can skip or let through and what bounds it."
  - "Fidelity: every Decision D1-D6 and every Constraint in `_lyx/discussion/decision-record.md` is carried by a card, and no card adds unlicensed scope. Check D3's `pull` classification, D4's shared check function, D6's `doc.go` updates (gitrepo, gitexec, fabricengine, battencli) and the `PATTERN-no-denied-recovery` and `PATTERN-hubforge-fixtures` constraints. The carry-over block in `## Open risks` licenses no scope."
---

Seed focus for round 1.
No finding is pre-stated; each entry names a place to look and a question.
