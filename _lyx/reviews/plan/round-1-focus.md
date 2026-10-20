---
round: 1
focus:
  - "Fidelity: walk every Decision (D1 onward) and every Constraint in `_lyx/discussion/decision-record.md` and settle which card carries it; report any Decision no card carries and any card scope the record does not license."
  - "Card order: the plan builds bottom-up across 23 cards; settle whether any card's `Uses` names a symbol a later card creates or re-signs, in particular cards 3-6 (replay seams), 9-10 (push retry and printing) and 15-18 (reconcile judge, moves, ended, seams) against the call-site cards 19-23."
  - "Verify coverage: settle whether every package any card targets appears in the `## verify:` command, including packages touched by cards 12-14 and 18-23 and any build-tagged hermetic tests."
  - "Attack-surface: card 12 (gate entry exemption), card 13-14 (local removal mode, no board write) and card 11 (detached push) loosen or route around guards; settle whether each card's `**Intent:**` states what the change lets through and what bounds it."
  - "Reachability of prescribed refusals: for each refusal, way-forward or stop branch in cards 6, 7, 9, 16 and 19-23, settle whether the card names an input and a test that reach it and whether an upstream check rejects that input."
  - "Test economy and granularity: settle whether cards 15-17 and the five call-site cards 19-23 plan per-symbol tests or tests duplicating existing coverage, and whether any card splits a private helper from the symbol it serves."
  - "Edit-card glyphs: for each `Edit` card whose `Intent` names specific symbols, settle whether targets are member glyphs and carry the re-sign arrow where parameters, results or receivers change."
  - "CLI shape: cards 10, 19 and 20 change `lyx` command output or flags (`fabric status`, `tender status`, `cleanup --apply`, `orch start`); settle them against the audience bullet of `pattern/PATTERN-cli-cobra.md`."
---

Focus covers the areas where a 23-card, bottom-up plan most likely fails: answer-key coverage, ordering across the replay and reconcile layers, and verify coverage.
