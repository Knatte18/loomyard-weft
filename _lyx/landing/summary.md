# Webster stop reaches the parent once, after the arbiter

A Webster stop that needs an answer now reaches the run's parent through one channel, the driver's stop chain, after the arbiter had its turn.
Before, Merriam sent its question to the parent by direct message and the driver then escalated the same stuck reason as a stop question, so one decision took several rounds.
A fresh Merriam started after a parent answer also did not know that answer.

Closes #875

## Changes

- **Master stencil** (`contracts/stencils/webster/webster-template-master.md`): every stuck rung of Merriam's failure ladder ends in `outcome: stuck` alone.
  The stencil states once that a batch Merriam cannot carry on, a refusal it cannot act on, or any stop of its run is never sent to its parent as a message; the driver carries the stuck reason to the arbiter and on to the parent.
  Merriam still messages its parent on the waiting rung and the decision rung.
- **Parent decisions in the Master prompt** (`internal/websterengine`, `internal/discussionparser`): `RenderMasterPrompt` fills a new required marker `parent_decisions` with each `Added after Discussion` entry of the decision record, heading and body verbatim, in record order, or `none` when the record has none or no record is named.
  The stencil tells Merriam these are answers its parent already gave: act on them like a decision notice, hand them to every fork, never ask about them again.
  `discussionparser`, the record's sole parser, gains `AddedDecisionTexts`, which reuses the span helper `dropAddedDecisionEntries` already used.
- **Arbiter first for Webster** (`internal/sheddriver`, `internal/loomcli`): `ArbiterSeam` gains an optional `JudgesFirst func(producer string) bool`, and the rule table a row after the hand-over and tackler-resume-returned rows.
  A blocked stop at a row `JudgesFirst` marks goes to the arbiter, skipping the tackler.
  `loomArbiterSeam` in `internal/loomcli` marks the Webster row only; darn and batten leave `JudgesFirst` nil, so their stops behave as before.
- **Docs**: `internal/websterengine/doc.go`, `internal/sheddriver/doc.go`, and `PATTERN-shed-stop-chain` in `PATTERN.md` describe the new path.

## Design decisions

- The driver's channel is the one kept, because it runs the arbiter first; Merriam's direct question is the one removed.
- The new row only skips the tackler.
  The landing-halt, budget-exhausted, precondition-unmet and pause rows still precede it, the arbiter's own eligibility check still applies, and an arbiter hand-on or refusal still hands back to the parent as an escalated stop does.
  The rule table stays recipe-blind: the Webster row name enters only through the seam.
- No question-text dedupe in the driver.
  A re-stepped Webster's Merriam now sees the recorded answer through `parent_decisions`, and the arbiter reads the decision record and hands on only a call no recorded decision settles, so the question does not recur.
  A text match would misfire on a new stop whose wording repeats.
- `Run` reads the added decisions before the declared PATTERN changes, so an unreadable decision record reaches its own refusal, which names `transient` and `lyx webster run` as the way forward.

## Out of scope

Stops of any row but Webster, the tackler's hand-back path for other rows, the recovery and cold verify-fixer prompts, and batten's driver.

## To check by hand

- Not run: a live Webster run that ends stuck.
  Confirm the orch receives one question, after an arbiter turn, and that a re-stepped Merriam's prompt lists the decision the orch recorded with `lyx loom decision add`.
- An arbiter that resumes Webster with a goto to its own row relaunches Merriam; a stuck reason no decision answers then stops again and costs a second arbiter spawn before the hand-on, bounded by `arbiter_max_spawns`.
