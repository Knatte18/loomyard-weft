---
round: 1
focus:
  - "Attack surface, D3 and D5: a goto onto an inlined row enters a block with no gate evaluated, and the skip target jumps past every inlined row. Settle whether each Decision states what the bypass or jump lets through and what bounds it, and whether `always` as the only shipped check leaves any gate path unbounded."
  - "Attack surface, D2 and D8: the expansion refusals and `profile_recipe` are new guards and a new routing key. Settle whether a sub-recipe or an `arras` task can reach a row, budget or profile block the bound says it cannot."
  - "Nesting, D2 against D6 and D4: with a block inside a sub-recipe, settle whether the rewrite of empty `on_done` to 'the block's exit' and the skip target are well defined for the inner and outer block, in what order they apply, and whether the block path and the Done condition in D4 agree."
  - "Consistency of D2, D3, D4 and D11: the block row's `on_done` is the sub entry after expansion and its skip target is the original exit. Settle whether D4's Done condition, D7's progress step from the block row to its exit, and the acceptance bullets all read the same edge."
  - "Deferred research: settle whether any Decision, Constraint, Open risk or acceptance bullet leaves a fact the design rests on to Plan, such as how the existing skip and entry-gate path routes, or how profile lookups key on recipe name, without the answer and its evidence recorded."
  - "Parent-only questions: settle whether Open risks and the question ledger (the `arras` name awaiting approval, the peer run `decision-record-on-board` adding `approve_seam` to the same Discussion-Bouncer row) hold a design conflict with another run that only the parent can settle, and whether any such item is classed as `decision` rather than left as a writer choice."
  - "Leanness and placement: settle whether `decision-record.md` holds deliberation narrative that belongs in `support-log.md`, and whether the support log's review-round bullets hold any requirement the plan writer needs but never reads. Extract a requirement into Decisions or Constraints before any relocation finding."
  - "Board rules: settle whether any Decision edits this task's own board entry, or files or promotes a task rather than a note, including the follow-ups named in Scope Out."
---

Seed focus for round 1.
Entries name where to look and which question to settle; none caps severity or states a verdict.
Optional "Notes for the plan writer", rejected alternatives in `decision-record.md` and call-site enumeration are left out by the rubric's `Do not flag` list.
