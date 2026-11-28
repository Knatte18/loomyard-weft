---
round: 4
focus:
  - "Card 12 (`_lym/plan/12-board-test-moves.md`), support in `boardengine_test`: does any helper it plans there repeat git-using or fixture support that stays in use in `fabricengine_test` (`recordsFixture`'s `primeWeft`, `originCommit`, `endEntryOnOrigin`, `originCommitFile`, `newRecordsFixture`, `newPairFixtureAt`) or that `hubforge`, `boardkit` or `gitkit` already provide? Where the card shares one copy instead, check that every caller of the old copy is in its targets, that the imports hold under `PATTERN-test-isolation` and `PATTERN-leaf-packages`, and that any edit to another concern's file squares with D9."
  - "Card 12, test kits and assertions: does any `internal/testkit` package or `gitkit` export it plans assert on the code under test, beyond `t.Fatalf` on its own setup or an exception `PATTERN-test-isolation` lists? Where the card has moved rows assert for themselves, check that each value they read is exported and reachable from `boardengine_test`, and that the card's targets and ImpactSummary agree with its text."
  - "Card 12, `OriginSeams.Teardown` and `OwnerOf` in the moved board tests: production fills both from `internal/pairteardown`. Does any planned stand-in or assertion rest on hand-written teardown or owner output (`PATTERN-test-stand-ins`, D12)? Where the rows use the real seams, check in the code that the teardown of these pairs runs in tier1 with no tmux server, what `machinename.ReadIdentity` answers in a test binary, which board fields the real owner reader needs, and that every asserted mutation and owner outcome is reachable that way."
  - "Card 12, coverage across the move: for each row it moves from `fabricengine`, does the planned setup still produce the state the row asserts, including heddle replayer registration for a row that changes `heddle.json`, and is any dropped assertion still held by a named test on another card?"
---

All four entries concern card 12, where the support code of the moved board tests has to satisfy
the test-isolation and stand-in rules while each moved row keeps what it asserts.
