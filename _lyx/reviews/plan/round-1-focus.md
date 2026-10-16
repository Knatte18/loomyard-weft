---
round: 1
focus:
  - "Fidelity: check that D3's refusal of a `top` strand with positional or sharing kind, its window-too-short reclaim order and its bottom-most-strand case are each carried by a card in cards 1 and 2, and that no card adds scope outside Scope or Out."
  - "Fidelity: check that D4's four refusals (unknown strand, no content declaration, wrong pane id, count below 1) and its unchanged-count no-op are carried by card 3, and that each has a named input and a named test that reaches it."
  - "Fidelity: check that every D7 time rule (folded repeats, goto entries, entries without history, removed rows, unreadable recipe) and every D8 rule (one append per process, `O_APPEND` single write, told path, failed append logged) is carried by cards 7 and 8, with the D8 rebase-onto-cleanup constraint and the Docs-in-same-commit constraint placed in some card."
  - "Attack-surface: check that card 3's content-rows operation, card 4's Selvage marker and bindings, card 10's keep-or-replace of the `runs` strand and card 12's shell line each state in `**Intent:**` what the change lets through and what bounds it."
  - "Card order and granularity: check that no card in 2 to 12 uses a symbol a later card creates, that card 6's move out of loom does not strand card 7's use of it, and that no card is one-per-symbol."
  - "Edit targets: check that each `Edit` card whose Intent names specific symbols lists member glyphs (not only file paths), carries the re-sign arrow where parameters, results or receiver change, and has an `ImpactSummary` with a real conclusion."
  - "Test economy and verify coverage: check for new tests that repeat behavior an existing test covers or that test one symbol each, and that the `## verify:` section runs the tests of every package the cards target, including `-tags integration` tests."
---

Seed focus only; the reviewer still reads every artifact.
`lens-review.md` sits in the plan directory but not in the Card Index, so it is not a card.
