# A Webster stop reaches the parent once, after the arbiter

A batch Merriam could not recover reached the orch on two channels at once.
Merriam asked its parent by message and held, and when Webster ended, the Go driver escalated the same stuck reason as a stop question.
Each re-stepped Webster then started a fresh Merriam that did not know the parent's earlier answer, so one decision took three rounds of messages (issue #875).

## What changes

**Merriam asks nothing about a stop.**
The Master stencil's failure ladder now opens with one rule: every rung that ends the run writes `outcome: stuck` and stops, with no message to the parent.
The run's driver carries the `stuck_reason` on.
Merriam messages its parent only on the `waiting` rung (a `NEEDS_PARENT` step) and when a decision arrives with no batch left.

**The arbiter judges a Webster stop first.**
`sheddriver.ArbiterSeam` gains an optional `JudgesFirst func(producer string) bool`.
The rule table gains one row: a blocked stop at a row `JudgesFirst` marks goes to the arbiter with no tackler call.
The row sits after the hand-over and tackler-resume-returned rows.
The landing-halt, budget-exhausted, precondition-unmet and pause rows still come before it, and the arbiter's own eligibility check still applies.
The loom CLI's driver wiring, which loom and darn share, marks `loomshed.NameWebster`; batten leaves the seam unset.
Before this change, the tackler handed a Webster stuck stop straight back with its own question, which skipped the arbiter.
Now a question reaches the parent only when the arbiter hands it on, as the orch role already assumes.

**A fresh Merriam is told the parent's answers.**
`discussionparser.AddedDecisionTexts` returns every `### Added after Discussion` entry of the decision record, heading and body verbatim.
It shares its entry extents with `IdentityText` through `addedDecisionSpans`.
`RenderMasterPrompt` takes the entries and fills a new required marker, `parent_decisions`, or `none` when there are none, under the stencil's parent-decision section.
The stencil tells Merriam that these are answers its parent already gave: it follows them like a decision notice, its forks inherit them, and it never raises a question one of them answers.
`lyx loom decision add`'s live notice still reaches a Merriam that is already running.
The prompt covers a Merriam that a re-step starts after the parent answered.

## Design decisions

- The driver does not dedupe questions by their text.
  The two deciders that could raise the question again, a fresh Merriam and the arbiter, now both see the recorded answer: the arbiter already reads the decision record.
  A text match would misfire on a new stop whose wording repeats an old one.
- Only the Webster row is marked.
  Sending every tackler hand-back to the arbiter would change batten's and darn's stop chains too, which is wider than the entry.
- The recovery and cold verify-fixer prompts get no decisions.
  The entry names Merriam, and Merriam passes decisions on through its forks.
- The orch role stencil needed no edit.
  It already says the driver hands a stop to the arbiter first, and this change makes that true for Webster.

## Docs

The `internal/sheddriver` package doc lists the new rule row and seam, and the `internal/websterengine` package doc states the single-channel rule and the `parent_decisions` rendering.
The `Geometry.DecisionRecordPath` comment names its new reader.

## Tests

- `TestDecideDrive`: the new row, the hand-over before it, and landing, budget and precondition halts at a marked row still handing back.
- `TestGoDriver_Settle`: a blocked stop at a marked row calls the arbiter with no tackler call, and its hand-on parks with one notice. An unmarked row still meets the tackler.
- `TestAddedDecisionTexts`: entry extents, order, a fenced heading, and a record with no entries, a missing record and an empty path.
- `TestRenderPrompts` and `TestTemplates_FillRequiresEveryRequiredMarker`: the decisions render in order under their section, `none` renders without decisions, and the marker is required.
