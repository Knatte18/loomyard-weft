---
round: 1
focus:
  - "Card order, cards 5-8 and 9-13: does any card use a symbol, seam or marker that a later card creates (the waiting marker, the deferrable gate, the `LoopSpec.OnStart` hook, the `tendercli.EndOwnedElsewhere` seam)? Name the card and the later card."
  - "Attack-surface, cards 6, 7, 12, 13 and 15: each new refusal, skip, defer or kill path (deferred gate, driver-check skip, `reed down` refusal, owned-elsewhere end, `tender reap`) states in its Intent what it can now skip or end and what bounds it, notably that `End` and `reap` never signal an unrelated or reused pid."
  - "Reachability of prescribed refusals and guard branches, cards 9, 10, 11, 12 and 15: each names an input and a test that reaches it, and no upstream check rejects that input first."
  - "Fidelity to the decision record: every Decision and Constraint, including D2, D4, D6 and the hibernate and shuttle decisions, is carried by some card, and no card adds scope outside it. Check the Shared Decisions reading of D6's 'ahead' (card 8) against the record."
  - "Granularity and test economy, cards 2, 9 and 10: private helpers and substrates listed as separate Create glyphs versus bundled into their surface's card; new tests that an existing test such as `TestTeardown_CallSequence` already covers."
  - "Edit glyphs, cards 9-17: where an Intent changes a member's parameters, results or receiver, the glyph carries the re-sign arrow; no Edit card lists only non-test source files while its Intent names symbols."
  - "PATTERN fit: the PATTERN changes in the overview against `PATTERN-run-process-owner`, `PATTERN-pair-teardown`, `PATTERN-leaf-packages`, `PATTERN-told-geometry` and `PATTERN-lyxdirs-single-declarer`; card 4's `LoopSpec.OnStart` hook against the `shedverbs` import allowlist."
  - "Research steps: any card that leaves a fact to be found while it runs, or says to halt on discovering one, especially in cards 1, 14, 16 and 17 (proc reads, hibernate confirmation, shuttle waiting)."
  - "Board-entry edits: no card changes this task's own board entry's body or scope."
---

Seeded from the overview, the Card Index and card 9.
The cards were not all read in full, so the focus lists the questions most likely to produce findings rather than known defects.
