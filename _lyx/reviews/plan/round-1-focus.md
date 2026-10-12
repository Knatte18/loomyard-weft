---
round: 1
focus:
  - "Cards 3 and 4 against D2 and D3 in the decision record: does each prescribed deny or pass-through input in the Intent have a named test input that reaches it, and does an upstream check or the `commandPositionPattern` head reject any of those named inputs before the new row sees it?"
  - "Card 7 against D6 and its acceptance criteria: is every D6 behavior (latch set and keep-stamp, input-box clear at any age, retry after a failed read, persisted dialog offset before the notifier call, offset advance after the notice, reattach rebuild, `QuietHold` and `HoldDeferral` not suppressing) carried by the Intent, and does every `TestWait_DialogNotice` row name an input that reaches its branch?"
  - "Card 7 granularity and test economy: it bundles the latch, notice, `RunState` field, parser row and four doc edits; is that one independently reviewable unit, and does `TestParseSessionSignals` or `hold_test.go` already cover behavior the new rows repeat?"
  - "Card 8 against the decision record's last acceptance criterion and open risk: does the planned command reach a prompt that cards 3 and 4 let through, and does the card's `Edit` list cover every file its Intent changes (the scenario file's header comment included)?"
  - "Card order and `## verify:` coverage: does any card use something a later card introduces (card 5 on card 3's deny table, card 8 on card 7), and does the section run the tests of every package the cards target, including the integration-tagged deny tests and `cmd/lyx`, in the order the plan template states?"
  - "Edit-card glyphs and re-sign arrows: for each `Edit` card, are the named symbols in the Intent listed as member glyphs, and does any listed member whose parameters, results or receiver the Intent changes lack the arrow?"
---

Focus covers the cards that add guards or new runtime edges (3, 4, 6, 7, 8), where the attack-surface and refusal-reachability questions bite, plus the structural checks (order, verify, glyphs) that apply to every card.
No entry restates a check the round's own gate already enforces.
