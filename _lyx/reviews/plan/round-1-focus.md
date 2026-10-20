---
round: 1
focus:
  - "Fidelity: check each Decision and Constraint in `_lyx/discussion/decision-record.md` against the cards, in particular the D4 dry-run handling that the Shared Decisions section reinterprets, and whether card 8 is licensed to pin the dry-run report."
  - "Card order: cards 5, 6 and 7 switch readers to `internal/lenses` and card 8 deletes burler's config; settle whether any reader card depends on something a later card builds."
  - "Verify coverage: settle whether every package any card targets, including packages card 8 touches through the burler deregistration and `burler.yaml` key moves, has its tests run by the `## verify:` section."
  - "Attack surface and reachability: for the never-prune rule in reconcile and the two key moves at first start after deploy, settle whether the card's `Intent` states what is skipped or let through and what bounds it, and whether each prescribed refusal has a named input and test that reach it."
  - "Test economy and granularity: settle whether cards 1 to 4 plan new tests that an existing yamlengine, configengine or configsync test already covers, and whether the per-name map plumbing is split into cards at the right unit."
---

Seed focus drawn from the plan overview and card index only.
