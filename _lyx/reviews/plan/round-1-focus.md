---
round: 1
focus:
  - "Cards 1 and 2 (and 3 and 4): every D1-D7 Decision and Constraint in the decision record is carried by some card, in particular D5's refusal-spec rows (repeated-conflict halt, Webster reroute-cap halt), the recipe header comment, and the pin that the recipe's fasit text equals the `websterengine` path declaration; settle which card carries each."
  - "Card 1: `ErrPlanDrifted` is listed under Delete; settle whether every remaining reference (record-batch, recover-batch, tests, stencil claims) is covered by an Edit target and whether a card placed earlier depends on a later one (Card order)."
  - "Cards 2 and 4: the new refusals and guards (begin-batch first-step removal, Run-entry removal after run.lock, repeated-conflict halt, stale record on an auto-rebaseline refusal) each need a named input and a named test that reaches it; settle reachability, and that no test only calls an error constructor."
  - "Cards 3 and 4: the new `on_reroute` edge and the last-routed file are routing changes; settle whether each card's Intent states what it can skip or let through and what bounds it (attack-surface question)."
  - "Cards 1-6: `Edit` cards that list only Go file paths while the Intent names symbols, and member glyphs missing the re-sign arrow where the Intent changes parameters, results or receiver."
  - "Cards 1-6: test economy; new test functions that duplicate coverage an existing test already gives, or one test per symbol."
  - "Card 6: the terminal guard is an injected seam in `configcli`; settle that the card's Intent names what it blocks and its bound, and that the `--print`/`--set` way forward and `PATTERN-cli-cobra.md` audience bullet hold."
  - "`## verify:`: settle that every package a card targets (websterengine, webstercli, shedadapters, loomcli, contracts/stencils, configengine, configcli, any loom recipe test) has its tests run, including integration-tagged ones, and that section order matches the plan template."
---

Focus is derived from the decision record's decisions against the six cards and the verify section; `lens-review.md` and `support-log.md` were not used.
