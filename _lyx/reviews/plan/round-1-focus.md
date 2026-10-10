---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1 to D9 and the In-scope doc updates: package `doc.go` files, `docs/overview.md`, `PATTERN.md`, `pattern/PATTERN-hub-store-gc.md`) and settle which card carries each; also settle whether any card's scope (for example card 2's `tools/deploy` edit or card 9's sync subject) goes beyond what the record licenses. Entries inside the `lyx:carry-over` markers do not count."
  - "Card order: cards 7, 8, 10 and 11 depend on `Bolt.PullThenCommitWritten` and `pullFirst`, card 10 deletes the `hubreconcile` `revisionLabel` copy that card 1 moves, and cards 4 and 6 read `Clean`/`Label` and the banner keys from cards 1 and 3. Settle whether each card's `Intent` names only symbols that an earlier card creates."
  - "Member glyphs and the re-sign arrow: for each `Edit` card whose targets are only Go file paths while the `Intent` names symbols, and for each member glyph whose `Intent` changes parameters, results or receiver (for example `reconcileOne`, `StencilSource`, `Topology`, `CommitSeededStencils`, `Mode`), settle whether the glyph or the arrow is missing."
  - "Verify coverage: settle whether `## verify:` runs the tests of every package any card targets, including `tools/deploy` and `cmd/lyx`, and whether the section order matches the plan template's."
  - "Attack-surface: for the seed-commit drop behind the destruction gate (card 7), the skip-when-Bolt-cannot-be-updated path (cards 7, 8, 10) and the gc skip under a live session (card 12), settle whether each card's `Intent` states what it can now skip or drop and what bounds it."
  - "Test economy: for each card that plans a new test function or file, settle whether an existing test or a public-surface test already covers the behavior, and name it."
  - "Granularity: settle whether any card bundles independently reviewable units, or splits a private helper from its type (for example the `Create` glyphs in cards 1, 2 and 5 against their consumer cards)."
  - "`ImpactSummary` on `Edit` and `Delete` cards: settle whether each carries a real blast-radius conclusion rather than a restatement of `Intent`."
---

Seed focus derived from the plan overview and cards 1 and 2 plus the head of the decision record; later cards were not read in full.
