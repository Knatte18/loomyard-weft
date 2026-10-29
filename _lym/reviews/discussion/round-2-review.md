---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md, Open risks bullet 2 and Acceptance criteria D3 real-session bullet; support-log.md Q11
    summary: Whether the terminal draws a ring wake's body under its creel mail row is a fact the D3 and D4 operator copy rests on, and the record leaves it to the smoke run with a rule to send the design back to discussion when the run finds it
    fix: Settle the fact now from Claude Code 2.1.312's bundled source, tracing how a queued task-notification (mode task-notification, built by the asyncRewake enqueue with a summary tag and a trailing body) is rendered in the transcript, or by a one-off hand run of an asyncRewake hook exiting 2 in a scratch session. Record the answer with its evidence in D4's rationale. If the body is drawn, delete Open risks bullet 2 and the clause "the run records whether the terminal draws the wake's body" from the D3 real-session criterion, and mark Q11 resolved. If it is not drawn, give the ring an operator copy in D3 or D4 now (for example the summary label carrying the claimed lines, or the lines in rewakeSummary rendered per ring through the seam) and drop the open risk.
  - id: F2
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D3 second paragraph and second Bound; Constraints bullet on delivery once
    summary: D3 resolves an acknowledgement failure after the write as silent loss (exit 0 when the daemon already journaled the read), the opposite of hook-read's visible-duplicate exception, and never weighs exiting 2 on an unconfirmed acknowledgement
    fix: In D3, state the choice between exit 0 and exit 2 for a written-but-unconfirmed acknowledgement and why the ring takes it (loss in the journaled-but-unconfirmed case against a duplicate in the not-journaled case), or switch to exit 2 on an unconfirmed acknowledgement and change the D3 Bound, the Constraints exception and the D3 hook-wait acceptance bullet to the duplicate instead of the loss. Add the alternative not taken to support-log.md's Rejected alternatives.
  - id: F3
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D3 second paragraph and the Notes bullet on the Go client
    summary: The existing creel client reconnects and resends a hook-wait whose acknowledgement write or confirmation loses its connection, which defeats D3's rule that a ring carrying mail never resumes the wait, and the record names only the client's auto-acknowledgement as the path to change
    fix: In D3's second paragraph add that hook-wait's call does not take the client's reconnect-and-resume path once a ring outcome carrying lines has reached its handler, so a connection lost at the acknowledgement or its confirmation ends the call; in the Notes bullet on the Go client name CallWatching's resume of a Wait request after a connLostError beside the auto-acknowledgement; and add to Scope's docs that internal/creelengine/doc.go's sentence on resuming a dropped hook-wait changes accordingly.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D5 outcome list
    summary: D5 names no outcome for a claim whose client acknowledged but whose read the daemon failed to journal, which is neither acknowledged nor released
    fix: In D5's outcome list add that case, for example ack-failed at Warn with strand, path and ids, replacing the daemon's existing journal-failure Warn for claims, and add it to the D5 acceptance bullet's outcomes.
---

The record meets its board entry: the entry's two Find bullets are researched and answered from Claude Code 2.1.312's source, its Open bullets are each settled or bounded, and the ring hands the mail over as the entry asks.
I checked the record's upstream claims against the installed 2.1.312 binary: Stop additional context is pushed into the continuation list unless the simple-mode guard holds, the continuation counts against `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`, and an asyncRewake exit 2 builds `<rewakeMessage> <stderr>` with the `rewakeSummary` label and priority `next`, with marker patterns replaced.
No PATTERN entry is violated: provider shapes stay in claudeengine, the ring adds no pane typing, the mark is a tolerated request field, and D5's levels follow the logger's Level policy.

### [MEDIUM] Ring wake's operator copy left to the smoke run

**Location:** `decision-record.md` Open risks bullet 2, the D3 real-session acceptance bullet, `support-log.md` Q11.

**Issue:** Half the Goal is that the operator sees the mail once.
For the ring, whether that holds rests on one upstream fact: does the terminal draw a task-notification's body under its summary row?
The record states it is unsettled and hands it to the tier3 run, with a rule that a negative answer returns the design to discussion.
The rubric names exactly this, a fact the design rests on left to be found later plus a halt handed downstream, as a finding.
The fact is researchable from the same bundled source the record already reads, or by one hand run.
Round 1's fixer deferred this part of its F4 because the minified source read did not find the body's rendering site.
That deferral does not settle the finding: the rubric counts a fact left for a later step as a defect, whatever stopped the source read, and a scratch session running an asyncRewake hook that exits 2 answers it directly without the source.
Whether the ring needs a different operator copy is a choice the writer can make once the fact is known, so this stays `design`, not `decision`.

### [LOW] Ring's acknowledgement-failure policy is the opposite of hook-read's, unweighed

**Location:** `decision-record.md` D3 and Constraints.

**Issue:** `hook-read` writes before it acknowledges, so its failure mode is a duplicate.
`hook-wait` under D3 exits 0 on any failed acknowledgement, so when the daemon journaled the read and only the confirmation was lost, the mail is lost as read (`internal/creelengine/server.go` `handOut` journals then confirms; `internal/creelengine/client.go` `awaitAckConfirmation` reports the lost confirmation).
Exiting 2 on an unconfirmed acknowledgement would turn that loss into at most a duplicate.
D3 states the loss as a Bound but never compares the two; the Rejected alternatives cover resuming the wait, not exiting 2.

### [LOW] The client's resume path defeats "never resumes the wait"

**Location:** `decision-record.md` D3 and Notes for the plan writer.

**Issue:** `Client.CallWatching` reconnects whenever the error is a `connLostError`, the connection was established and the request waits, and `resume` resends a `VerbHookWait` request unchanged.
The acknowledgement write and `awaitAckConfirmation` both return `connLostError`.
So with D3's handler in place, a dropped confirmation re-registers the waiter with the lines already on stderr; the next ring appends more and exits 2, and the model gets the first set again while the released mail is handed out once more.
`internal/creelengine/doc.go` states the resume as the rule.
The D3 acceptance bullet would catch it, but the record names only the auto-acknowledgement as the client change, so the plan writer has no pointer to this path.
Three lenses raised it; kept once.

### [NIT] D5 has no outcome for a journal failure after acknowledgement

**Location:** `decision-record.md` D5.

**Issue:** In `handOut`, a client can acknowledge while `Reading.Ack` fails to journal; the daemon logs a bare `Warn` and answers an error.
That attempt is neither `acknowledged` nor `released`, so D5's "one line per attempt with strand, path, ids and outcome" leaves it without its line.

## Rejected

- bound-breaker F1, the same client-resume defect as F3, filed as `consistency`: merged into F3 as `design`, since an existing path that undoes a decision is a design finding about it.
- walkthrough F1, the same client-resume defect: merged into F3.
