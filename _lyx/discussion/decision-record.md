# Decision record — webster-stop-asked-once

## Goal

A Webster stop that needs an answer reaches the run's parent once, through the run's stop chain, after the arbiter had its turn, and an answer the parent recorded reaches the Merriam session that a later re-step starts.

## Scope

- The Master stencil (`contracts/stencils/webster/webster-template-master.md`): Merriam reports a batch it cannot carry on only through `outcome: stuck`, never by a message to its parent.
- The Master prompt: Go renders the run's parent decisions, the decision record's `Added after Discussion` entries with their text, into a new required marker, so a fresh Merriam knows every answer its parent recorded.
- The shed driver (`internal/sheddriver`): a blocked stop at a row the arbiter seam marks as judged first goes to the arbiter, skipping the tackler.
- The loom driver wiring (`internal/loomcli`) marks the Webster row so.
- Docs: `internal/websterengine/doc.go`, `internal/sheddriver/doc.go`, and the orch role stencil where it states the escalation path.

Out of scope: a stop of any row but Webster, the tackler's own hand-back path for other rows, the recovery and cold verify-fixer prompts, and batten's driver.

## Decisions

### Merriam never asks its parent about a stuck run

Decision: every stuck rung of the Master stencil's failure ladder ends in `outcome: stuck` alone; the stencil says outright that a batch Merriam cannot carry on, a refusal it cannot act on, or any stop of its run is never sent to its parent as a message, and that the run's driver carries the stuck reason to the arbiter and on to the parent.
Merriam messages its parent only on the waiting rung (a `NEEDS_PARENT` step) and the decision rung (no batch left for a decision).
Rationale: the incident's double channel was Merriam's direct question plus the driver's stop question over the same stuck reason; the driver's channel is the one that runs the arbiter first, so it is the one kept.

### A fresh Merriam is told every parent decision of the run

Decision: `RenderMasterPrompt` fills a new required marker, `parent_decisions`, from the decision record at `Geometry.DecisionRecordPath`: each `### Added after Discussion` entry, heading and body, verbatim, in record order, or the literal `none` when the record has none or standalone mode names no record.
The `discussionparser` package, the record's sole parser, gains the read that returns the entries with their text.
The stencil tells Merriam these are answers its parent already gave, to act on as on a decision notice: hand them to every fork it spawns and never ask about them again.
Rationale: the resumed Webster in the incident started a Merriam that did not know the parent's earlier answer; `lyx loom decision add` is where the orch records an answer to a design call, and the live notice reaches only a Merriam already running.

### Webster's blocked stop goes to the arbiter before the tackler

Decision: `sheddriver.ArbiterSeam` gains an optional `JudgesFirst func(producer string) bool`; the rule table gains a row after the hand-over and tackler-resume-returned rows: a blocked stop whose row `JudgesFirst` marks goes to the arbiter.
The loom driver wiring sets it to the Webster row (`loomshed.NameWebster`); darn and batten leave it nil.
Bound: the row skips only the tackler, for blocked stops at marked rows; the landing-halt, budget-exhausted, precondition-unmet and pause rows still precede it, the arbiter's own eligibility check still applies, and an arbiter hand-on or refusal still hands back to the parent exactly as an escalated stop does.
Rationale: the board's "arbiter tries first" decision; the tackler's hand-back today sends its question straight to the parent, which the orch rules already assume never happens.

### The parent's answer is not re-asked

Decision: no question-text dedupe in the driver.
A re-stepped Webster's fresh Merriam gets the recorded answer through `parent_decisions`, and the arbiter already reads the decision record and hands on only a call no recorded decision settles.
Rationale: both deciders that could raise the question again now see the answer, so the question does not recur, while a text match would misfire on a new stop whose wording repeats.

## Constraints

- `PATTERN-stencil-ownership`: the decisions text is rendered by Go into the prompt; Merriam still reads no decision record.
- `PATTERN-sole-parsers`: only `internal/discussionparser` parses the decision record.
- `PATTERN-shed-driver`: the rule table stays recipe-blind; the row name enters only through the seam.
- `PATTERN-shed-stop-chain`: rule table, tackler, arbiter, then parent stays the order for every stop the new row does not mark.
- `PATTERN-test-speed`: new behaviour is pinned as table rows in existing tests where one covers the behaviour.

## Auto-mode assumptions

- The parent records an answer to a Webster stop with `lyx loom decision add`, as the orch role's design-call bullet says.
- A Webster stuck outcome reaches the driver as a blocked stop at the Webster row, since the row has no `on_stuck`.

## Open risks

- An arbiter that resumes Webster with a goto to its own row re-launches Merriam; a stuck reason no decision answers then stops again and costs a second arbiter spawn before the hand-on, bounded by `arbiter_max_spawns`.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-2-review.md; fixer report: _lyx/reviews/code/round-2-fixer-report.md.
  - judges-first-webster-only-marking-unpinned (class design, severity MEDIUM, open in rounds 1, 2)
  - run-decision-record-refusal-untested (class design, severity BLOCKING, open in rounds 1, 2)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- The rule table sends a blocked stop at a row `JudgesFirst` marks to the arbiter, and a blocked stop at an unmarked row, or with no seam wired, to the tackler as before.
- A landing halt, a spent bounce budget and an unmet precondition at a marked row still hand back.
- The master prompt renders each added decision's heading and text under `parent_decisions`, and `none` for a record without one and for an empty record path.
- The added-decision read returns each entry's text up to the next heading and skips fenced headings.
- The Master stencil's stuck rungs name no message to the parent, and the stencil states the stuck-reaches-the-arbiter rule once.

## Notes for the plan writer

- The master stencil's `parent_decisions` marker joins the required set in `stencil.FillOptional`'s call in `RenderMasterPrompt`; the stencils package's marker tests list each stencil's markers.
