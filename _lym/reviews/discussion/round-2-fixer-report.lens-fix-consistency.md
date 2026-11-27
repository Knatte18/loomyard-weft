```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that each changed passage still agrees with the decisions, criteria, table rows and cross-references it touches.

Changed text checked against its neighbours: D3's tackler bullet, the new "count nothing" bullet and the D3 criterion; D4's lead-in, the bounce-budget bullet, the lift-resume and hold-over-a-live-entry text and the D4 criteria; the D5 no-wait sentence; D6's `budget` causes, its rule order, the board note, the spent continue, the Finalize hold, the Bound and the D6 criteria; D7's pane relay and unattended hold sentence and its Review criterion; D8's dedup and reminders and their criteria; the D9 review-round bullet; D10's chase key and button-off post and the D10 criterion; D12's warning; the notes for the plan writer.

**MEDIUM: D7's new question sentence contradicts D9 and D2.**
- Changed text: D7, "for an agent question it relays as severe, it answers the agent that the operator decides, which ends that question's chase".
- D9 says that under `unattended` "An agent's question is answered and never relayed".
- D2's table row for agent questions agrees with D9: the `unattended` orch column reads "answers, never relays (D9)", and its operator column lists only "a run that cannot finish within its record".
- So D7 now describes the unattended orch relaying a question as severe, which D9 and D2 forbid.
- Either D9 and the D2 row need an exception for a question whose answer is the severe class, or the D7 clause should be dropped.

**MEDIUM: the hard-cap rule's place in D6's ordered list contradicts its own wording.**
- Changed text: D6, "The arbiter settles a `budget` escalation by these rules, in order", with the hard-cap rule ("is handed on in place of the continue") second, ahead of the green-and-deferred accept rule.
- "In place of the continue" says only the continue is replaced at the hard cap, and `circlingVerb` never refuses `accept` (`internal/loomcli/circling.go:76`).
- Read in order, though, the second rule hands on every `budget` escalation at the hard cap before the accept rule is reached. That includes a green, all-deferred round under `true`, which the accept rule would otherwise accept.
- The Bound ("never at or past the hard cap") limits only the continue, so the Bound sides with the wording and against the order.

**LOW: the support log still says the arbiter files the board note.**
- Changed text: D6, "the driver, acting on that accept, files one board note … and the arbiter's settings gain no board grant".
- `support-log.md:75`, in Rejected alternatives, still reads "only the arbiter's budget accept past the scope files one note per run (D6)". The note is now filed by the driver, not the arbiter.

**LOW: the new "count nothing" bullet matches settle records by rule text, against D3's own matching rule.**
- Changed text: D3, "matched by its settler, its action and, for a `go-rule` record, its rule: … a `go-rule` `restep` under the interrupted-reinvoke rule".
- The `go-rule` record's rule field is the decision's text ("an interrupted step under the reinvoke policy: re-step", `internal/sheddriver/driverules.go:296`).
- D3 says elsewhere that a settle record "gains the stop's class" and the count "matches a settle record by its row and class".
- D4 now allows only one row, the bounce budget's, to match on text.
- Matching the interrupted-reinvoke re-step by its class, not its rule text, would agree with both.

Tool calls made beyond my context:
- One Bash call: `git diff` on the decision record, to see the change set (it showed no diff, since the file is not tracked under `_lym`), and a grep for line positions of the D2 table rows, the `settle: operator` sites and the old D3 and D8 wording, to check sibling text without re-reading the file.
