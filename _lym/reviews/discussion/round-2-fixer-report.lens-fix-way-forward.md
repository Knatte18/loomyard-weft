```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward lens: checking that every refusal, error, next step, row or test the fix introduced can be reached and makes progress.

Ways forward the fix introduced:
1. D3: a written but unconfirmed acknowledgement exits 2. Mail the daemon did not journal "reaches the model once more at a later claim".
2. D3: a failed write of the lines or of the acknowledgement exits 0, and the bell's unanswered-ring window follows.
3. D6: the echo is handed out at the strand's next tool-boundary or Stop claim. It stays owed after a release.
4. D6: a Stop with an echo and no mail exits 1, and the merged Stop command records the turn end.
5. D5: the `ack-failed` outcome at `Warn`.
6. D6 acceptance criterion and the revised D3 and D5 acceptance criteria.

Findings:

- **MEDIUM**, decision-record D3: "mail the daemon did not journal stays unread and reaches the model once more at a later claim". The Bound "an acknowledgement whose confirmation is refused or lost shows the mail the daemon did not journal twice … once at a later claim" and the Constraints exception make the same claim.
  Summary: when the daemon refuses the acknowledgement, the mail it did not journal stays reserved, so no later claim can hand it out and the promised second showing never happens.
  Evidence: `Reading.Ack` (`internal/creelengine/verbs.go:471-486`) deletes reservations only up to the failing id. On a failed journal write it returns early, so later ids stay in `store.reserved` and `r.ids` is not cleared. `handOut` (`internal/creelengine/server.go:470-472`) logs the error and calls no `Release`. `reserved` is cleared only by `Ack`, `Release` and the pane-path helpers (`deliver.go:729,741`). Claims skip reserved ids (`pool.go:446`), and the strand flag still stands, so the mail stays unread and unclaimable until the daemon restarts.
  D3's release rule ("every way the ring's answer path ends before the daemon journals the acknowledgement releases the claim") does not plainly cover a journal failure after the acknowledgement. D5 even calls `ack-failed` "neither acknowledged nor released".
  The record needs to say that the ring path releases what an `ack-failed` left unjournaled. Otherwise the refused-confirmation branch leads nowhere for that mail.

- **LOW**, decision-record D3: "Once the acknowledgement is written, `hook-wait` exits with the wake code whether the daemon confirms it, refuses it…". This sits beside the unchanged "The ring's own claim marks the ring answered at the acknowledgement".
  Summary: on a refused acknowledgement the session wakes with the mail, but the ring stays unanswered. The 30-second bell for an unanswered ring can still fire and type a bell line, even though the model already has the mail.
  Evidence: D3 ties answering to the journaled acknowledgement, and D3 itself says an unanswered ring falls to "the bell's 30-second window". The changed text names no rule that stops this for the refused-but-woken case.

Tool calls:
- One grep for `reserved` in `internal/creelengine/*.go`. My context had `Ack` and `Release` but not the other places that clear a reservation, and I needed them to confirm the reservation is never cleared after `ack-failed`.
