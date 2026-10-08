---
round: 1
focus:
  - "Card 11 (reset-targets): `Intent` is the card that lets `report-head` through while the run lock is held. Does it state what that skips and each bound D1 names: highest begun batch, no live recovery strand, report on disk, descends from `StartSHA`, non-terminal?"
  - "Cards 12 and 14 (reset-start-archives, retire-fresh-after-reset): the archive-only path of `reset --to start` drops several refusals. Does the card state what it lets through and what bounds it: no git run, record moved and never deleted, lock, merge and foreign-branch refusals kept, recovery strands removed first?"
  - "Cards 16 and 17 (amended-cards-rebaseline, amended-cards-recovery) and card 19 (rebaseline-card-index): each widens what rebaseline accepts, and card 17 adds a ladder rung that re-runs recovery. Does each `Intent` state the bound: in-flight batch only, `--card` named only, at most one forced recovery per amendment, overview change confined to the Card Index?"
  - "Card 6 and the Master ladder: the `config_invalid` stop rung. Does the plan state that it fires only on the parse-or-validation case and never on an unreadable file?"
  - "Fidelity: walk every D1 to D13 Decision and every Constraint in `_lyx/discussion/decision-record.md` against the cards and name any that no card carries; also name any card scope the decision record does not license. Check the Shared Decisions against the record, in particular `reset-archive-is-the-fresh-archive`, which departs from D3's wording."
  - "Card order: do cards 11 to 14 use `ResetPairCode`, the keep-reset (card 10) or the archive factoring (card 12) before the card that adds it? Does card 17 depend on card 16's `AmendedCards` and card 19 on card 18's planparser function? Name the card and the later card it depends on."
  - "Verify coverage: the `## verify:` section runs `go test ./...` and `go test -tags integration ./...`. Does every package a card targets, `tools/tokencount` and `internal/loomshed` included, have its tests run, and is the section in the order the plan template states?"
  - "Test economy: for cards that plan new test functions (reset targets, the refusal-way-forward rows, the batcher migration, the computed base), does an existing table test already cover the behavior, or does a card plan one test per symbol? Name the existing test or the public-surface test."
  - "Granularity: do cards 2, 3 and 4, and cards 8 and 9, split a private helper or an inseparable constructor into a card of its own? Does each public-surface symbol that is reused across packages have its own card?"
  - "`ImpactSummary`: on the `Edit` and `Delete` cards that change a shared symbol (`resetKeepStep` deletion, `master_base` retirement, `Batch`'s signature), is it a real blast-radius conclusion and not a restatement of `Intent`?"
  - "`Custom` and `Prosa` typing: for any `Custom` card, could its mixed targets be a multi-label combination of the other types? For any `Prosa` group, is every target a file or unit self glyph?"
---

Seed focus for round 1.
Every entry names a place to look and a question to settle, and none pre-states a verdict.
I left out the checks the format gate already enforces, dependency-field completeness, and `ImpactSummary` on `Rename`, `Move`, `Prosa` and `Custom` cards.
