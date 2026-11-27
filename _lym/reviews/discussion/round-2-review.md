---
verdict: BLOCKING
findings:
  - id: F26
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:131
    summary: "D3 counts a tackler resume from its tackler attempt on the false premise that the tackler's attempts are kept apart from the driver's settle records; `tackle` in internal/sheddriver/drive.go writes a `SettleRestep` settle record with settler tackler for every resume, so one tackler-resumed halt counts two or three times (halt entry, restep record, attempt) and a class reaches `settle: operator` at its second stop."
    fix: "In D3, replace the tackler bullet: a tackler resume counts through its settle record (settler `tackler`, action `restep`, written by `tackle` in `internal/sheddriver/drive.go`), which names its halt entry like any settle record and counts with it as one stop; the tackler attempt counts nothing; and a restep record of a stop D3 excludes (an arbiter settle, an interrupted-reinvoke re-step after a wake) counts nothing, matched by its settler and action. Drop the clause that the tackler's attempts are kept apart from the settle records. Add to D3's first tier0 criterion a halt entry resumed by the tackler that counts once."
  - id: F27
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:211
    summary: "D6 rests on the false premise that a `budget` escalation holds an open BLOCKING finding or a red live gate; the Bouncer also escalates `budget` when it cannot read the round's review, ledger, previous decision, escalation record or live state, and with an unreadable ledger and a green gate D6's accept test (every open BLOCKING finding deferred) holds vacuously, so under `true` the arbiter accepts a round whose findings nobody read, dropping the arbiter's rule to decide nothing on an unreadable input."
    fix: "In D6, replace the sentence `A budget escalation holds an open BLOCKING finding or a red live gate` with the causes the Bouncer escalates `budget` on (an open BLOCKING entry, a red or unreadable live gate, an unreadable review, ledger, previous decision or escalation record; `settleUndecided`, `softCapRule`, `blockingEscalationDetail` and `liveRedDetail` in `internal/shedadapters/bouncer.go`), and add a first rule: an escalation whose detail names an input that could not be read is handed on at every level, with neither accept nor continue, as the arbiter stencil's rule says today. Add to D6's second criterion a `budget` escalation over an unreadable ledger with a green gate under `true` that is handed on."
  - id: F28
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    carries: F1
    location: _lym/discussion/decision-record.md:230
    summary: "D6's bound that nothing lands a round with an open BLOCKING finding under `unattended` still breaks at Finalize: `lym loom circling accept` on a run halted after its fix reviews ran out of rounds (`finalizeCirclingVerb` in internal/loomcli/circling.go) lands the fixer's change with its open findings appended, and D6 and D9 give the unattended orch a hold rule only for review escalations."
    fix: "In D6, extend the unattended rule and its Bound to the Finalize halt after its fix reviews ran out of rounds: under `unattended` neither the arbiter nor the orch runs `lym loom circling accept` there while an open BLOCKING finding or a red gate remains, and the orch holds the run as D6 says. Say the same in D9's review-round bullet, add a Finalize fix-review halt under `unattended` to D6's second criterion, and name it in the D6 Review criterion on `role-autonomy.md`."
  - id: F29
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    carries: F3
    location: _lym/discussion/decision-record.md:260
    summary: "While the dobby's orch has its slack button off, the orch relays in its pane only a halt whose notice reads `settle: operator`; D10's severe posts after a chase (a `settle: orch` halt, an agent question, a dead driver) go only to `<shortname>:SLACK`, which drops them as handled (internal/slackengine/outbound.go postMessages), so under `unattended` those stalls reach neither Slack nor the pane."
    fix: "In D10, state that the chase line the dobby sends its orch for a stall it will escalate also carries the stall's key, and that while the dobby's orch has its slack button off the orch relays in its pane each D10 severe post's stall as it relays a `settle: operator` halt (D7). Widen D7's pane-relay sentence to name D10's posts, have D12's warning name them, and add the button-off case to D10's first criterion."
  - id: F30
    severity: MEDIUM
    class: design
    origin: lens:interactions
    carries: F4
    location: _lym/discussion/decision-record.md:277
    summary: "D10 says D8's reminders cover a stall it posts as severe, but D8's reminders run until the run leaves the halt; an agent-question stall and a dead-driver stall leave the child running with no halt, so their reminders have no defined end and either never stop or never start."
    fix: "In D8, state that a reminder of a severe post ends on the end event of what it reminds of: for a halt, the run leaving that halt; for a D10 stall, the events D10 lists that end its chase (the question's answer or settle, the driver's revive or a transition of the run). Add a question stall and a dead-driver stall to D8's second criterion, each reminded and then stopping at its own end event."
  - id: F31
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F9
    location: _lym/discussion/decision-record.md:219
    summary: "D6's per-segment continue reads any decision file with decision `continue` and cause `budget` as spent, but the Bouncer's soft-cap automatic continue writes exactly that file (`recordAutoContinue` in internal/shedadapters/circling.go, flagged automatic), so after one auto-continue the arbiter never continues; D6 also never says what follows when `lym loom circling continue` refuses a `budget` escalation at or past the hard cap, and the arbiter's `lym board upsert` grant for kind note alone is unpinned."
    fix: "In D6: (1) count as the segment's spent continue only a `round-<n>-circling-decision.md` with decision `continue`, cause `budget` and the automatic flag false, the flag `readCirclingFile` reads; (2) state that a `budget` escalation at or past the perch's hard cap, where `circlingVerb` refuses the continue, is handed on; (3) either name the settings rule that limits the arbiter's `lym board upsert` to kind note and the test that pins it, or have Go file the note after the arbiter's accept. Add an auto-continued segment and a budget escalation at the hard cap to D6's second criterion."
  - id: F32
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F11
    location: _lym/discussion/decision-record.md:171
    summary: "The pause record keeps one live entry per scope, and a pause of a scope that already has one adds no entry (internal/tenderengine/pause.go, `Added`), so the orch's hold writes no hold reason when the slug already holds the operator's pause or an adopt's bolt-hold entry; once that entry lifts (the dobby lifts a bolt-hold itself), D4's lift resumes a run the orch holds."
    fix: "In D4, state how a hold is recorded over a scope that already has a live entry: the hold sets its hold reason on that live entry of the slug's scope, and the dobby's lift of a bolt-hold entry carrying a hold reason replaces it with a hold entry of its own, never resuming the run. Add both cases (a hold over the operator's slug pause, a hold over a bolt-hold entry) to D4's pause criterion."
  - id: F33
    severity: MEDIUM
    class: design
    origin: lens:interactions
    carries: F12
    location: _lym/discussion/decision-record.md:167
    summary: "D4 has the dobby resume a pause hand-back once its pause lifts but names no verb or path; `liftPending` in internal/dobbyengine/passes.go gives a pass only to a drivable run, and a resume through `lym loom resume` waits up to the keep-warm cap under D5, whose no-wait option D5 grants only to batten, so a dobby tick can block on the park wait against D5's claim that none does."
    fix: "In D4, name the path the lift resume takes (for example the run's batten pass, which D4 makes the lift give to a run handed back for that pause, and which passes D5's no-wait option on its child resume), and state that it never waits on the park. In D5, name that path beside batten's child start and resume as a caller that passes the no-wait option. Add the lift resume of a run whose driver has not parked to D4's pause criterion."
  - id: F34
    severity: LOW
    class: design
    origin: lens:bound-breaker
    carries: F16
    location: _lym/discussion/decision-record.md:273
    summary: "D8's keyless post dedups on the slug alone, so a keyless orch relay of a distinct halt under `true`, such as a scope change or a cancel, posted within `slack.remind_after_min` of a keyed post for another halt of the same run, notifies no phone, against D8's own rule that a new halt of the same run notifies again."
    fix: "In D8, state that a keyless post dedups only against an earlier keyless post of the same slug, and a keyed post against an earlier post of the same slug and key. Add to D8's first criterion a keyless post that follows a keyed post of the same slug within the window and mentions the operator."
  - id: F35
    severity: LOW
    class: consistency
    origin: lens:interactions
    carries: F21
    location: _lym/discussion/decision-record.md:317
    summary: "Under `unattended` the orch still relays two classes itself, a run that cannot finish within its record and a step only the operator can take, while the run stays halted under its `settle: orch` key; D10's chase ends on no relay, so one interval after the chase the dobby posts the same halt as severe again, and a question the orch relays is posted again under `question-<id>`, breaking D7's one severe post per halt."
    fix: "In D7, state that under `unattended`, once the orch relays a halt it classifies as severe, it holds the run as a hold (D4), which ends D10's chase; and that for an agent question it relays as severe it answers the agent that the operator decides, which ends that question's chase. Add to D10's first criterion an orch-relayed halt under `unattended` that gets no second severe post."
  - id: F36
    severity: LOW
    class: design
    origin: lens:bound-breaker
    carries: F23
    location: _lym/discussion/decision-record.md:161
    summary: "D4 says every new row's class is raised as a named kind or field so no rule matches on message text, but says the bounce budget's hand-back already marks it, and that mark is the error-text prefix `shedengine.ReasonBounceBudgetExhausted` that `budgetExhausted` in driverules.go and `arbiterstop.Check` match."
    fix: "In D4's bounce-budget bullet, state that the row matches the `shedengine.ReasonBounceBudgetExhausted` reason prefix as today's rule table does, the one prefix row it keeps, or name the field the hand-back gains; keep the no-message-text rule for the other rows."
prior:
  - id: F1
    status: open
    answer: "The review-escalation paths are fixed; the Finalize fix-review accept still lands open findings unattended, carried as F28."
  - id: F2
    status: resolved
  - id: F3
    status: open
    answer: "The pane relay covers `settle: operator` halts; D10's escalated posts are still dropped with the button off, carried as F29."
  - id: F4
    status: open
    answer: "D10 now chases the question and the dead driver; the reminders it relies on have no end for them, carried as F30."
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
    answer: "D3 now names each entry; the tackler bullet's premise is false, raised as the new F26."
  - id: F9
    status: open
    answer: "The record D6 names also holds the Bouncer's automatic continues, carried as F31."
  - id: F10
    status: resolved
  - id: F11
    status: open
    answer: "The hold reason is not written when the scope already has a live entry, carried as F32."
  - id: F12
    status: open
    answer: "Batten passes no-wait; D4's lift resume is a dobby path the option does not reach, carried as F33."
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: open
    answer: "The `halt:` key settles keyed posts; a keyless post still dedups against keyed ones, carried as F34."
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
  - id: F20
    status: resolved
  - id: F21
    status: open
    answer: "Fixed for a halt with no settling action open; the other classes the orch relays still get a second post, carried as F35."
  - id: F22
    status: resolved
  - id: F23
    status: open
    answer: "The other rows name their kind; the bounce-budget row's mark is a text prefix, carried as F36."
  - id: F24
    status: resolved
  - id: F25
    status: resolved
---

### [BLOCKING] D3 counts a tackler-resumed halt two or three times

**Location:** `_lym/discussion/decision-record.md:131`

**Issue:** D3 counts a tackler resume "from its tackler attempt, since the tackler's attempts are kept apart from the driver's settle records".
That premise is false.
`tackle` in `internal/sheddriver/drive.go` appends a `SettleSpawn` record and, on a resume, a `SettleRestep` record with settler `tackler` and outcome "tackler resume".
D3 also counts "a driver settle record whose rule retried the row".
So one blocked halt the tackler resumes counts through its halt entry, its restep record and its attempt, unless the restep record names the halt entry, and even then twice.
A row's third halt then comes at the first or second real stop.
That stamps `settle: operator`, posts a severe halt to the operator's phone, and ends lym's retries early.
This breaks D3's Bound and the declared `PATTERN-stop-settlement` rule that a lym retry counts toward the row's halt count, which means it counts once.
Three lenses found this independently.

### [BLOCKING] D6's budget accept rests on a false premise and can accept an unread round

**Location:** `_lym/discussion/decision-record.md:211`

**Issue:** D6 says "A `budget` escalation holds an open BLOCKING finding or a red live gate" and builds its accept rule on that.
`settleUndecided`, `softCapRule` and `blockingEscalationDetail` in `internal/shedadapters/bouncer.go` also escalate `budget` in two other cases:
- when the round's review, ledger, previous decision or escalation record cannot be read;
- when the live state cannot be read, through `liveRedDetail`.

With an unreadable ledger and a green gate, D6's first rule holds vacuously: no open BLOCKING finding can be found that is not deferred.
Under `true` the arbiter then accepts a round whose findings nobody read.
The arbiter's stencil today says to decide nothing at cause `budget` and hand on, naming the input that could not be read (`loom-template-arbiter.md`).
D6 replaces that rule for every budget escalation and does not keep it for this case.
By the rubric's attack-surface rule, D6 opens a new accept edge whose stated bound does not hold.

### [MEDIUM] Finalize's fix-review accept still lands open findings unattended

**Location:** `_lym/discussion/decision-record.md:230`

**Issue:** Carries F1.
`finalizeCirclingVerb` in `internal/loomcli/circling.go` settles a run halted at Finalize after its fix reviews ran out of rounds.
It marks the fixer's unapproved change accepted, appends the open findings, and lands the change on resume.
D6's unattended rule and Bound name only review escalations, and D9 defers to D6.
So the orch under `unattended` may still run this accept, against the Bound "nothing lands a round with an open BLOCKING finding … nor the orch".

### [MEDIUM] With the slack button off, D10's escalated posts reach no one

**Location:** `_lym/discussion/decision-record.md:260`

**Issue:** Carries F3.
`postMessages` in `internal/slackengine/outbound.go` drops a message as handled when its orch's button is off.
D7's pane exception covers only a notice reading `settle: operator`.
D10's severe post after a chase can be for any of three stalls, and none of them carries that stamp:
- a `settle: orch` halt;
- an agent question;
- a dead driver.

So under `unattended` with the button off, the stalls D10 escalates are dropped on Slack and never relayed in the pane.

### [MEDIUM] D8's reminders have no end for a question or dead-driver stall

**Location:** `_lym/discussion/decision-record.md:277`

**Issue:** Carries F4.
D10 has D8's reminders cover the severe post of a stall, but D8 ends a reminder only "until the run leaves the halt".
A question stall and a dead-driver stall have no halt, since the child is running.
So whether those reminders stop is undefined.

### [MEDIUM] D6's spent-continue test matches the Bouncer's automatic continue

**Location:** `_lym/discussion/decision-record.md:219`

**Issue:** Carries F9.
`recordAutoContinue` in `internal/shedadapters/circling.go` writes a decision file with decision `continue`, cause `budget` and the automatic flag set.
D6 reads exactly that shape as "the segment's continue is spent", so the arbiter never continues in a segment where the soft-cap rule continued once.
Two further gaps fall in the same D6 rule:
- **A budget escalation at the hard cap.** Under `unattended`, `BlockingEscalates` runs first in a window the hard cap opened, so such a round escalates `budget`, and D6 never says what follows when `circlingVerb` refuses the continue there.
- **The arbiter's board-note grant.** That grant of `lym board upsert` "for that kind alone" names no settings rule or test that limits it to a note.

### [MEDIUM] The orch's hold can fail to write its hold reason

**Location:** `_lym/discussion/decision-record.md:171`

**Issue:** Carries F11.
The pause record holds at most one live entry per scope, and a pause of a scope that already has one adds no entry (`internal/tenderengine/pause.go`).
Two cases follow:
- **The operator's own pause.** A hold over a slug the operator already paused writes no hold reason, so lifting the operator's entry lets D4's lift resume the held run.
- **An adopt's bolt-hold entry.** A hold over that entry writes no hold reason either, and the dobby lifts a bolt-hold entry itself once the adopt clears, with the same result.

### [MEDIUM] D4's lift resume names no path and can block a dobby tick

**Location:** `_lym/discussion/decision-record.md:167`

**Issue:** Carries F12.
D4 has the dobby resume a run handed back for a pause once that pause lifts, but names neither the verb nor the path.
`liftPending` in `internal/dobbyengine/passes.go` gives a pass only to a drivable run.
A resume through `lym loom resume` waits up to the keep-warm cap under D5, and D5 grants the no-wait option to batten alone.
That contradicts D5's claim that no dobby or batten tick blocks on the wait.

### [LOW] A keyless severe post is muted by a keyed one

**Location:** `_lym/discussion/decision-record.md:273`

**Issue:** Carries F16.
"A post with no key dedups on the slug alone" mutes a keyless orch relay of a distinct halt, such as a scope change under `true`.
That happens when the relay follows a keyed post for another halt of the same run within `slack.remind_after_min`.
It contradicts D8's "a new halt of the same run … notifies again".

### [LOW] An orch-relayed severe halt is posted again by D10

**Location:** `_lym/discussion/decision-record.md:317`

**Issue:** Carries F21.
Round 1's fix moved only "no settling action open" to D10's post.
Under `unattended` the orch still relays two classes itself, while the run stays halted under the same `settle: orch` key:
- a run that cannot finish within its record;
- a step only the operator can take.

No relay ends D10's chase, so the dobby posts the same halt again two chase intervals after it began, outside D8's mention window by default.
An agent question the orch relays is posted again under its `question-<id>` key.

### [LOW] The bounce-budget row still matches message text

**Location:** `_lym/discussion/decision-record.md:161`

**Issue:** Carries F23.
D4 says no rule matches on message text, and that the budget's hand-back "already marks it".
That mark is the reason prefix `shedengine.ReasonBounceBudgetExhausted`, which `budgetExhausted` in `internal/sheddriver/driverules.go` and `arbiterstop.Check` read with `strings.HasPrefix`.

## Rejected

- Walkthrough W2 and bound-breaker BB1, the spent-continue test, raised as BLOCKING: kept as F31 at MEDIUM. The defect costs the arbiter its one continue and hands the round on, which never lands anything, and it carries round 1's F9.
- Walkthrough W4, the second severe post, raised as BLOCKING: kept as F35 at LOW. It is round 1's F21 still open, and a carried finding keeps or lowers its severity.
- Walkthrough W3 and existing-paths F-c, the budget premise, raised as MEDIUM: kept as F27 at BLOCKING, following bound-breaker BB2. The vacuous accept opens a new accept edge whose bound does not hold.
- Walkthrough W5, driver hand-backs that match no table row: these are precondition unmet, interrupted under the handback policy, an error with no kind, and an awaiting stop with a parent notice. Not raised. D2 already gives every unlisted hand-back `settle: orch`, so the verdict is defined, and a new non-BLOCKING finding cannot be raised in an incremental round.
- Interactions 7, D2's "every hand-back to the orch carries `settle: orch`" read against D7's `settle: operator` notice: not raised. The sentence's subject is the other stops, the ones lym does not settle and the table does not make severe, and a wording-level LOW cannot be raised as new in an incremental round.
- Walkthrough W7, existing-paths F-d and bound-breaker BB3, the budget escalation at the hard cap: merged into F31.
- Bound-breaker BB10, the unpinned `lym board upsert` grant: merged into F31.
- Walkthrough W6, existing-paths F-f, interactions 3 and bound-breaker BB7, the lift resume path: merged into F33.
- Walkthrough W9 and interactions 4: merged into F30. Walkthrough W10 and interactions 5: merged into F29. Walkthrough W8, interactions 6 and bound-breaker BB9: merged into F34.
