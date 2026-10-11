---
round: 1
focus:
  - "Fidelity: walk every Decision D1-D11, every acceptance criterion and the open risks in `_lyx/discussion/decision-record.md` against the cards, and settle whether each is carried by some card; check in particular the `Table.Validate` refusals (Required, Accept or seat gate on the chair, Accept on a non-required advisor), the `lyx burler run` caller move, the `contracts/stencils/stencils.go` registry re-keying, and the doc.go updates for burlerengine, seatengine, shuttleengine and burlercli."
  - "Card order: settle whether card 7 (retire `fix_start`, `Engine.Run` edits) and card 8 (round rewrite) can each build and pass tests at their position, and whether card 8 or 9 depends on anything a later card supplies (card 9's stencils against card 8's marker values, card 10's removal against card 9, card 11 against cards 7 and 8)."
  - "Attack surface: for the new hold-deferral predicate (cards 1 and 5), the required advisor with `Accept` (card 4), per-seat fork authorization (card 3) and the removed `await-review` verb (card 10), settle whether each card's `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Reachability of refusals: for each refusal or guard branch a card prescribes (unknown seat, not-stopped, skipped-handoff, review-changed-after-fix, `Validate` refusals), settle whether a named input and a named test reach it and whether an upstream check rejects that input first."
  - "Edit-card glyphs: on cards 3, 4, 5 and 8, settle whether member glyphs carry the re-sign arrow wherever `Intent` changes a member's parameters, results, type parameters or receiver, and whether any Edit card lists only Go file paths while naming symbols."
  - "Test economy and granularity: settle whether card 8's large test set and the seat-engine cards' new tests repeat behavior an existing test covers (name that test), and whether any card is split per literal symbol or any public-surface symbol is buried in another card."
  - "Verify coverage: settle whether every package any card targets, including `shuttlecli`, `seatengine`, `shuttleengine`, `loomcli`, `shedrecipe` and the `contracts/stencils` package, has its tests run by `## verify:`, and whether the section's order matches the plan template."
---

Seeded from the overview and cards 7, 10 and 11 plus the decision record; cards 1-6, 8 and 9 were not read in full.
