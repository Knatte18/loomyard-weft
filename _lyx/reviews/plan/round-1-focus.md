---
round: 1
focus:
  - "Card order across `00-overview.md`: cards 4 to 8 rewrite commands whose `Use`, `Short` and `Long` card 1 and card 2 define; check each rewrite card's `Uses` against cards 1, 2 and 3 and settle whether any card depends on a later one, card 14 on card 13 included."
  - "Fidelity to `_lyx/discussion/decision-record.md`: settle whether every Decision and Constraint (D1 to D7 and the intake-comments and strand-path items) is carried by some card, and whether any shared decision (`upsert-issues-key`, `index-note-annotation`, `help-mounted-through-set-help-command`) licenses scope the record does not."
  - "Attack-surface question on cards 2, 9, 12 and 14: the `lyx help` verb, `set-deps` id addressing, comment carry-through in `intake import`, and the watcher spawning `<path> help index`; settle whether each card's `**Intent:**` states what the change can let through and what bounds it."
  - "Edit-card glyphs and the re-sign arrow on cards 1, 2, 3, 9, 10, 11, 12, 13 and 14: settle whether any `Edit` card targets only Go file paths while its `Intent` names symbols, and whether any member whose parameters, results or receiver change lacks the arrow."
  - "Granularity of the form-rewrite cards 4 to 8: settle whether card 4 bundles eight packages into one reviewable unit and whether cards 6 to 8 split along independently reviewable lines, and whether card 3's `shedverbs.VerbText` change is a public-surface symbol that deserves its own card."
  - "Test economy on cards 1, 2, 9, 14 and 16: settle whether a new test duplicates the tree-walk form test in card 16 or the coupling test in card 9, naming the existing test that already covers the behavior."
  - "Verify coverage in `## verify:`: settle whether every package a card targets, including the `tools/` code, `cmd/lyx`, the skill location of card 15 and any nested module, has its tests run, and whether the section follows the plan template's order."
  - "`ImpactSummary` on `Edit` and `Delete` cards: settle whether each carries a real blast-radius conclusion and not a restatement of `Intent`, mainly for card 3's shared verb bodies and card 9's `set-deps` change."
---

Seeded from the overview and card index only.
The focus entries name where to look and leave the verdicts to the reviewer.
No entry targets a `Do not flag` item: dependency fields, `ImpactSummary` on non-`Edit`/`Delete` cards and mechanically gated checks are excluded.
