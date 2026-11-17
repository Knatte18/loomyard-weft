---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: "_lym/plan/11-stalemate-settlers.md"
    summary: "Card 11 says loomcli doc.go has one arbiter line to update, but the file enumerates the arbiter's admitted causes in two sentences"
    fix: "In Card 11's Intent, replace the phrase `loomcli doc.go's arbiter line` with both sentences by content: the one listing the stops the verb's closed list admits, which ends `a pending escalation of cause hard-cap, circling or decision`, and the one saying the table `asks the arbiter at an awaiting stop with a pending hard-cap, circling or decision escalation only`. Both gain `stalemate`. The targets need no change, since the card already lists `internal/loomcli/doc.go` under `**Edit:**`."
---

### [LOW] Card 11 names one arbiter line in loomcli's doc.go where two sentences enumerate the causes

**Location:** `_lym/plan/11-stalemate-settlers.md`

**Issue:** The Intent says "Its doc comment, arbiterstop's doc.go and loomcli doc.go's arbiter line list the admitted causes with `stalemate`".
`internal/loomcli/doc.go` holds two such enumerations: the verb's closed list ("a halted `awaiting` stop whose Bouncer row has a pending escalation of cause `hard-cap`, `circling` or `decision`") and, in the table's description, "it asks the arbiter at an awaiting stop with a pending hard-cap, circling or decision escalation only".
An implementer that reads "line" literally updates one and leaves the other saying the arbiter settles three causes.
The file glyph is already under `**Edit:**`, so the scope is licensed and only the Intent's wording is short.

## Rejected

- Card 2, `ParseFixerReport` and unknown keys: not a defect.
  The Intent has `ParseFixerReport` refuse any top-level key other than `dispositions` and `spawned`, as `checkReviewKeys` does for the review, which keeps the agent-written frontmatter strict as `PATTERN-tolerant-records` and the decision record's Constraint require.
  `TestCheckFixerReport` has a named row for a misspelled `dispositons` key that asserts the stray-key `way forward:` clause, which only that check produces, so the row reaches the refusal.
  The refusal sits in the parse and does not depend on the review's findings, so it also holds on a round whose fixer owns no finding, such as a gate-findings-only fixer, where an empty list would otherwise pass.
  The fix table appends `FixerReportGateEntry` for every spawned fixer, so that round is gated.
- Card 1, a tracked file matching an ignore pattern: not a defect.
  The throwaway index is seeded with `git read-tree HEAD`, so every tracked file is staged whatever the ignore rules say, and `git add -A` then re-stages its edits; ignore rules only exclude untracked paths.
  The id and `ChangedPathsBetween` both see the edit, and the card names a row for a tracked file matching an `info/exclude` pattern.
- Card 11, the arbiter's new reach: not a defect.
  The `**Intent:**` has a `Bound:` paragraph stating that the arbiter may accept a stalemated segment with findings still open and without the run's parent, and what bounds that: the accept goes through `acceptRound`'s `CarryOver` seam where the recipe sets `carry_over`, `budget` stays refused, and the segment stays Awaiting until an `accept` or `continue` is recorded.
- Card 10, `TestBouncer_Stalemate` input reach: every row's input can be set up through `Call`.
  Nothing in the Bouncer's `Call` path reads the fixer-report, so an unparseable report, an incomplete one and one carrying `spawned: false` reach `roundUnchanged` unrejected.
  A replay over an existing stalemate escalation record enters `settle` through `standsAsRecorded`, then `settleUnconverged` and `settleUndecided`, where `stalemated` answers true before any judge runs, with `StalemateRounds` at 2 or 0.
  The soft-cap rows reach `settleUndecided` through `judgeCall`'s harvest, so `spawned` is true there, and a ledger whose only open design-wrong entry is LOW makes `designWrongKeys` empty, so the check yields to `softCapRule`.
- Escalation-cause enumerations, other than the loomcli doc.go sentences above: each is covered.
  `valid`, `writeEscalation`, `readEscalation`, `readCirclingFile` and `ErrEscalationMalformed` are Edit members of Card 10, `internal/shedadapters/doc.go` is under its `**Edit:**` and not `**Extends:**`, `docs/overview.md` is under Card 12's `**Edit:**`, and the arbiterstop doc, the orch role and the arbiter stencil are under Card 11.
  escalation.go's file header and the comment over its cause constants lie outside every member glyph's span, so no listed glyph covers them, but Card 10's Intent names both for rewriting.
  Carrying such header text through the Intent is how this plan already handles `validate.go`'s header, and the file glyph cannot join the members under `redundant-file-target`, so this is not raised.
