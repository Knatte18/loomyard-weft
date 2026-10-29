```yaml
lens: fix-proposal
outcome: reported
```

Checked the fixes that followed the reviewer's proposals: F2 (exit-2 option), F3 and F4.

**MEDIUM: F2's duplicate-over-loss rationale assumes a release that the refused-acknowledgement path never makes.**
- **Changed text:** `decision-record.md` D3: "Once the acknowledgement is written, `hook-wait` exits with the wake code whether the daemon confirms it, refuses it on a failed journal write, …" and "mail the daemon did not journal stays unread and reaches the model once more at a later claim". The rewritten Constraints exception says the same: "mail the daemon did not journal is handed out once more".
- **Case:** a ring claims ids 1, 2 and 3. Then `Reading.Ack` fails on id 2. In `internal/creelengine/verbs.go:471-486` the loop has journaled id 1 and deleted id 2's reservation without journaling it. It returns before id 3, so id 3 stays in `store.reserved` and `r.ids` is not cleared. `handOut` (`server.go:470-472`) then writes the error and never calls `Release`. The record agrees: D5's new `ack-failed` is "neither acknowledged nor released".
- **Result:** after the exit 2, id 3 does not reach the model "once more at a later claim". It stays reserved and unclaimable until the daemon restarts. So the refused-acknowledgement case does not land on the visible duplicate the proposal traded for. Part of the mail is stranded instead, and nothing in D3 releases it.
- **What the fix needs:** D3's release rule ("every way the ring's answer path ends before the daemon journals the acknowledgement releases the claim") must cover a failed journal write explicitly, including the ids left after the failure point. Otherwise the refused case needs its own stated outcome. D5's `ack-failed` line and acceptance bullet would then need to say what happens to the reservation.

**F3 and F4: no breakage found.**
- F3: a connection lost before the outcome reaches the handler still resumes. D3 releases the claim, so a fresh ring follows with no duplicate. A loss at the acknowledgement's write exits 0, and Claude Code drops stderr at exit 0, so nothing is shown twice.
- F4: `ack-failed` is a sound outcome on its own terms. Its only interaction is the reservation gap above.

Tool calls:
- One grep of `internal/creelengine` for the bell's unanswered-ring window and the `handOut` callers. My context lacked them; I needed them to test whether exit 2 after a refused acknowledgement also lets the bell fire. Lines 274-277 of `deliver.go` suppress the bell while a turn runs, so I raised no finding there.
