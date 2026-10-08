---
round: 1
focus:
  - "Fidelity: check that each decision D1–D9 and every Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and that no card adds scope the record does not license; D7 (gate quiescence) and D9 (Webster-Review stays solo) are the likeliest to be dropped or widened."
  - "Granularity: cards 4, 5, 6, 7 and 11 each declare several new unexported helpers or types; settle whether any is a private supporting type that belongs in a neighbouring card, or a public-surface symbol that wrongly shares one."
  - "Member glyphs and re-sign arrows: for every `Edit` card whose `Intent` changes the parameters, results or receiver of a named function (cards 3, 6, 7, 8, 9, 10, 11, 13), settle whether the member glyph is listed and carries the re-sign arrow."
  - "Test economy: cards 1–14 each plan test files or functions; for each new test, settle whether an existing test already covers the behaviour or whether it is one test per symbol."
  - "Card order: cards 8 and 9 read `readRoundUsage` from card 7, and card 10 and cards 11–13 build on earlier cards; settle whether any card depends on something a later card introduces."
  - "Attack-surface: cards 3, 8, 10 and 12 introduce a fallback, a judge-controlled exclusion and new config keys; settle whether each `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Verify coverage: settle whether every package a card targets, including `internal/shedadapters`, burler, shuttle and the loom config package, has its tests run by the `## verify:` section, and whether the section's order matches the plan template."
---

Seed focus for round 1, set from the Card Index, the shared decision and card 7, with the decision-record headings as the answer-key outline.
No entry repeats a check the round's own gate enforces, and none relies on `support-log.md`.
