---
round: 1
focus:
  - "Cards 4, 5 and 7 (symbol and field renames typed `Edit` with an old/new table): does the Edit type fit, or is each table a set of existing-symbol renames that belong in a `Rename` group per the type table in the spec? Settle this once, as a `design` question about the method, not per card."
  - "Card 2 (`rollbackAdd` origin deletion): does each Intent bullet (proof type, ownership kind, remote-dirtiness kind, three-valued probe outcome, lease on the pushed commit) match D2 and its acceptance criteria exactly, and does its bound cover SkipGit/SkipPush, adopted branches and a pre-existing origin branch at or behind HEAD? Check the Coverage list against each of those cases."
  - "Card 1 and Card 2 ordering and file overlap: Card 2 relies on Card 1's push seam on `Topology`, and both edit `add.go`. Is the sequencing sound, and does Card 2's Coverage inject through the seam Card 1 defines?"
  - "Card 3 (prime records-only push): does the Intent carry every D3 route (`PushPairAnchored`, `Fabric.Commit`, the `sync` verb, `lyx config`) and the prime-name resolution failure rule? Does the Coverage name a test for each route?"
  - "Test economy in Cards 1, 2, 3 and 9: do any planned new tests duplicate an existing test's behavior? Name the existing test for any finding."
  - "Ordering of renames against file edits: Cards 3, 7 and 1 edit `weftgit.go` and `weftwiring.go` before Card 8 renames `weftgit.go`, and Card 9 edits `recordsgit.go` after it. Do the Edit lists and the Rename pairs line up with that sequence? Does each rename card's Edit list include every file that spells an old name, and do Cards 4 to 8 leave any `vocabExempt` spelling unrenamed that Card 10 would then fail on?"
  - "Fidelity to the decision record: is every D4 mapping row, every D5 bullet and D6 carried by some card, and does any card add scope the record does not license?"
  - "Attack-surface statements: for each hard cut and each removed or weakened guard (Cards 2, 3, 6, 7, 10), does the Intent state what it lets through and what bounds it?"
  - "`## verify:` section: does it run the tests of every package any card targets, in the template's order?"
---

The focus list names where to look and which question to settle.
It does not pre-state a verdict.
