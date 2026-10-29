---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:104
    summary: D6 states no echo rule for an acknowledgement whose journal write failed, on the ring or on a claim that hands out an echo
    fix: In D6, add one rule for a claim whose acknowledgement fails to journal. For a claim that hands out an echo, state that the echo is dropped at that acknowledgement, because the hook showed it before the client acknowledged. For a ring, state that the echo keeps the lines of the ids the daemon journaled read before the failure and nothing for the ids D3 released, which reach the operator when a later claim hands them out. Say in D5 that an `ack-failed` line lists the echo ids under the same rule. Add the case to the D6 acceptance bullet.
---

### [LOW] D6's echo has no rule for an acknowledgement whose journal write fails

**Location:** `_lym/discussion/decision-record.md:104` (D6), with D5's `ack-failed` at line 95 and D3's acknowledgement-failure rule at lines 65-70.

**Issue:**
D6 keeps the ring's echo "when the daemon journals a ring's acknowledged read" and drops a handed-out echo "at the claim's acknowledgement", and keeps it at a release.
An acknowledgement the client wrote and the daemon failed to journal is neither a journaled read nor a release, and D3 makes it a real case: the wake exits 2, so the model has all the lines.
`Reading.Ack` (`internal/creelengine/verbs.go:471`) journals the ids in order and returns at the first failure, so some ids may already be journaled read.
Two outcomes are left to the plan writer:

- On a ring, the ids journaled before the failure were shown to the model and are never claimed again, but D6 keeps an echo only for a journaled acknowledged read, so the operator may never see them. That breaks the Goal's "operator once" on the one path where the duplicate was accepted only for the model.
- On a claim that hands out an echo, the hook printed it before the client acknowledged, so dropping it shows it once and keeping it shows it twice.
  D3 accepts a visible duplicate for the mail, but D6 does not say whether the echo follows.

The choice is the writer's, and the plan writer never reads `support-log.md`, so the rule belongs in D6.
The fix states the rule that matches D3's ordering: shown means dropped, and unjournaled mail reaches the operator when it is handed out again.

## Rejected

- Focus 1, the ring wake's operator copy: no finding.
  D4 records the 2.1.312 evidence that the terminal draws the notification as one row with the `rewakeSummary` label and never its body, and that a per-wake summary is closed to a settings hook.
  D6 gives the operator one echo, and the D3 acceptance bullet checks one `creel mail` row with no mail under it and the mail once.
  No fact is left to the smoke run, and no rule sends the design back to discussion on a negative finding; the remaining Open risks are version drift and model weighting, which are not facts the design rests on.
- Focus 2, the acknowledgement-failure exit choice: no finding.
  D3, the Constraints exception and the D3 acceptance bullets agree: exit 0 when the write of the lines or of the acknowledgement fails, so the mail stays unread and the ring unanswered; exit 2 once the acknowledgement is written, whether the daemon confirms, refuses or the confirmation is lost.
  The loss-against-duplicate consequence is stated in D3, and `Reading.Ack` does leave the ids after a failing one reserved today, as D3 says.
- Focus 3, the client resume path: no finding.
  `exchange` in `internal/creelengine/client.go` returns a `connLostError` from both the acknowledgement write and `awaitAckConfirmation`, and `CallWatching` resumes any such error for a `Wait` request, so D3's rule needs a code change.
  The rule is reachable: a ring with lines arrives as an `AckRequired` response, which reaches the handler before the acknowledgement, so the client can return a non-resumable error from that point.
  Scope names the `creelengine/doc.go` sentence and `creelcli/doc.go`, and the Notes name `CallWatching`; listing `client.go`'s `resume` comment in Scope is call-site enumeration, which the rubric leaves to the plan writer.
- Focus 4, D5's outcome list: no finding.
  A claim the client acknowledged and the daemon failed to journal has `ack-failed` at `Warn`, a lost confirmation after the journaled read is `acknowledged`, and every end of the ring's answer path before the acknowledgement is `released`.
  The refused-claim and unreachable-socket cases are bounded out in D5 itself.
