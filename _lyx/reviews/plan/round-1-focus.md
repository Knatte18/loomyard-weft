---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` against the Card Index and confirm a card carries each; in particular the D8 and D9 landing-hold and Finalize decisions across cards 5, 13, 14, 15, 16 and 17."
  - "Card order: cards 10 to 12 (`orchcli.BeamUp`, `lyx beam start`) and cards 15 to 17 (pawl engine, recipe rows, wiring) name symbols other cards define; check that no card uses a symbol only a later card creates, for example `pawlengine.NameBeamAwait` in card 19 and `hubgeom.PawlPaths` in card 15."
  - "Reachability of refusals: cards 14, 18, 20 and 23 prescribe halts and refusals (`landHaltReason`, `memberHoldRefusal`, cancel refusing while members remain, no slot without a hold); check each names an input and a test that reaches it, and that no upstream check rejects that input."
  - "Attack-surface: cards 5, 7, 18, 20 and 22 add holds, skips, a land-request file and a cancel path; check each `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Member glyphs and re-sign arrows: Edit cards that change signatures (cards 4, 10, 14, 25, including `cancelOwnedElsewhere` and `ownerHaltReason`) must list member glyphs with the arrow where parameters, results or receivers change."
  - "Test economy: cards 15, 16 and 17 each plan new tests over the pawl rows; check for overlap with existing tests or per-symbol tests where one public-surface test covers the behavior."
  - "Research steps and board edits: look for any card that leaves a design fact to be found out while it runs, and any card that changes this task's own board entry's body or scope or files or promotes a task."
---

Seed focus drawn from the plan overview and a grep of the cards; each entry names where to look and the question to settle, with no verdict stated.
