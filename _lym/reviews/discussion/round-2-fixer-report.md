---
dispositions:
  - id: F26
    disposition: fixed
  - id: F27
    disposition: fixed-differently
    reason: "The proposed cause list named an unreadable escalation record, but escalateWith returns a Stuck for it (internal/shedadapters/bouncer.go:1670-1671), so no such escalation reaches the arbiter; and a detail-only test misses softCapRule's escalations, which carry no detail (bouncer.go:1505-1541, 1655-1657). D6 drops the escalation record from the causes and also hands on a budget escalation with no cause detail."
  - id: F28
    disposition: fixed
  - id: F29
    disposition: fixed-differently
    reason: "The orch cannot relay a post the relay dropped, since it never sees it (postMessages in internal/slackengine/outbound.go). With the button off, the dobby also sends its orch the severe post's line, and the orch relays that stall in its pane. The chase line carries the stall's key as proposed."
  - id: F30
    disposition: fixed
  - id: F31
    disposition: fixed
  - id: F32
    disposition: fixed
  - id: F33
    disposition: fixed
  - id: F34
    disposition: fixed
  - id: F35
    disposition: fixed-differently
    reason: "Answering a relayed question 'the operator decides' contradicts D9's never-relay rule and D2's row, and leaves the agent to halt into a second severe post. D7 relays only a question that shows a run cannot finish within its record, as that run's severe halt, holds the run, and answers the agent that it is held. D9 names that exception."
  - id: F36
    disposition: fixed
---

# Round 2 fixer report: discussion

Every change is in `_lym/discussion/decision-record.md` unless named otherwise.

## Changes

- **F26.** D3's tackler bullet now counts a tackler resume through its `tackler`/`restep` settle record, which names its halt entry and counts with it as one stop.
  The tackler attempt and its `spawn` record count nothing.
  The false premise about attempts kept apart is gone.
  A new "count nothing" bullet excludes settle records that retried nothing or settle an excluded stop, matched by settler, action and class.
  The D3 criterion now has a tackler-resumed halt that counts once.
  The plan-writer note that pointed at `tacklerrecords.go` is corrected.
- **F27 (fixed differently).** D6 now lists the Bouncer's `budget` causes with their functions.
  It says an unreadable escalation record ends as a Stuck.
  A first rule hands on, at every level, an escalation that names an unread input, has no cause detail, or has inputs the arbiter cannot read.
  The Bound adds "only with every input read".
  The D6 criterion adds the unreadable-ledger, green-gate case under `true`.
- **F28.** D6 extends the unattended hold and its Bound to the Finalize halt after the fix reviews ran out of rounds (`finalizeCirclingVerb`).
  D9's review-round bullet, the D6 criterion and the D6 Review criterion name it.
- **F29 (fixed differently).** D10's chase line carries the stall's key.
  With the slack button off, the dobby also sends its orch the severe post's line, and the orch relays it in its pane.
  D7's pane sentence, D12's warning and pane line, the D7 Review criterion and the D10 criterion name D10's posts.
- **F30.** D8's reminders now cover each severe post and end on its own end event: the run leaving the halt, or the end event D10 lists for the stall.
  The D8 criterion adds a question stall and a dead-driver stall.
- **F31.**
  - Only a non-automatic `continue`/`budget` decision file spends the segment's continue.
  - A `budget` escalation at or past the hard cap is handed on in place of the continue.
  - The driver files the board note after the arbiter's accept, through `internal/boardengine`, and the arbiter gets no board grant.
  - The support log's Rejected alternatives line now says the driver files the note.
  - The D6 criterion adds an auto-continued segment and a hard-cap `budget` escalation.
- **F32.** D4 says a hold over a slug with a live entry sets its hold reason on that entry.
  The dobby's lift of a bolt-hold entry that carries a hold reason replaces it with a hold entry and never resumes the run.
  The D4 criterion adds both cases.
- **F33.** D4's lift gives the run a dobby pass even though the run is not drivable.
  The pass resumes it with D5's no-wait option.
  The lift is kept in the dobby's state record until the resume succeeds or the run moves on, so a not-parked refusal is retried.
  D5 names the lift's resume among the no-wait callers.
  The D4 criterion adds the lift resume of a driver that has not parked.
- **F34.** D8 now dedups a keyless post only against keyless posts of the slug, and a keyed post against the same slug and key.
  The Bound requires the `halt:` key on every post of a keyed halt.
  The D8 criterion adds a keyless post after a keyed one.
- **F35 (fixed differently).** Under `unattended`, the orch holds a run once it relays its halt as severe.
  It relays a question only when the question shows the run cannot finish within its record, and then holds the run and answers that it is held.
  D9 states that exception.
  The D7 Review criterion and the D10 criterion add these cases.
- **F36.** D4's lead-in exempts the bounce budget from the kind-or-field rule.
  Its bullet says the row matches the `shedengine.ReasonBounceBudgetExhausted` prefix, as `budgetExhausted` and `arbiterstop.Check` do.
  The D4 criterion says the same.

## Self-check

- fix-way-forward, MEDIUM, the lift retry could not be reached because `liftPending` is cleared and lives only in memory: fixed. D4 keeps the lift in the state record and gives the pass to a run that is not drivable.
- fix-way-forward, LOW, "the operator decides" leaves the agent nothing to act on: fixed, merged with the D7/D9 finding below.
- fix-way-forward, LOW, the unattended orch had no rule for an escalation over an unread input: fixed. D6's orch holds it and names the escalation's way forward.
- fix-consistency, MEDIUM, and fix-proposal, MEDIUM, the D7 question sentence contradicts D9 and D2: fixed. D7 now covers a question that shows the run cannot finish, and D9 names that exception.
- fix-consistency, MEDIUM, and fix-proposal, MEDIUM, the hard-cap rule's place in the list blocked an allowed accept: fixed. The hard-cap rule moved into the continue rule.
- fix-consistency, LOW, the support log still said the arbiter files the note: fixed.
- fix-consistency, LOW, and fix-proposal, LOW, the interrupted-reinvoke exclusion matched rule text: fixed. It now matches by the class the record carries.
- fix-claims, MEDIUM, an unreadable escalation record never reaches the arbiter: fixed. It was dropped from the causes, and D6 says it ends as a Stuck.
- fix-claims, MEDIUM, and fix-proposal, MEDIUM, `budget` escalations with no detail slipped past the unread-input rule: fixed. A `budget` escalation with no cause detail is handed on.
- fix-claims, LOW, "batten pass" and the bolt-hold lift named no real path: fixed. D4 names `spawn` in `passes.go` and `syncBoltHold` in `catchupwait.go`.
- fix-proposal, LOW, a keyless orch reply about a keyed halt mentions again: fixed. D8's Bound requires the key on every post of a keyed halt.

## Disputed

None.

## Deferred

None.

## Rulings

None.
