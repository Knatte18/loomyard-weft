---
round: 1
focus:
  - "Fidelity: check that every Decision and Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and that no card adds scope the record lists under Out (editing a claimed board entry, new `darn.yaml` keys, splitting darn into rows)."
  - "Card order across cards 2-7: cards 5 and 6 use the `Env.DiscussionHistory` seam, cards 4 and 5 use the check from card 3, and card 3 uses card 2's history reader. Settle whether any card uses a symbol a later card creates."
  - "Verify coverage: `## verify:` runs `go test ./...` and `go test -tags integration ./...`. Settle whether every package a card targets, including non-Go stencil and contract targets, is covered, and whether the section follows the stencil's order."
  - "Attack-surface and refusal reachability on cards 3, 4, 6 and 7: the history-drop refusal, the commit at first clean validation, and `decision add` on a darn run. Settle whether each card states in `**Intent:**` what the change lets through and what bounds it, and whether each prescribed refusal names an input and a test that reach it."
  - "Edit-card glyphs: on Edit cards naming specific symbols in `Intent` (cards 2-7, 10-12), settle whether targets are member glyphs with the re-sign arrow where parameters, results or receivers change."
  - "Test economy on the new gate and history tests: settle whether an existing discussionparser, loomshed, fabricengine or shedrecipe test already covers the behavior a card plans a new test for."
---

Seed focus only; each entry names where to look and leaves the verdict to the reviewer.
