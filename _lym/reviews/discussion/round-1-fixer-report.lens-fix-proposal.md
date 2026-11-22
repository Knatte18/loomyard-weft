```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each review proposal the fixer carried out as proposed, looking for a concrete case where it goes wrong or breaks another part of the record.

1. **MEDIUM — D5, "Every settlement of a halt under autonomy records one `lym loom decision add <slug> --by parent` entry whose title names the row" (F10's proposal).**
   - Problem: this puts bookkeeping into the run's decision record. That record is a design input that downstream rows read and act on.
   - Case: an orch grants a spent bounce budget with `lym loom resume`. It must now add an entry such as "Code-Review: resumed after spent budget". That entry then reaches drift and darn as a decision about the run.
   - Evidence: the prompt's parent directive says "darn and drift give precedence over the body to added entries alone". `internal/driftengine/judgesnapshot.go:370-374` checks that the record only gains added entries (`CheckAddedEntriesSubset`) to decide whether the judge snapshot still holds. Each `decision add` also mails the run "Parent decision recorded…", which it acts on as an `ly:orch` message.
   - Fix direction: the review's alternative, counting from a record that every settling verb writes, avoids this.
2. **MEDIUM — D4, "that recorded value governs the review to its end, whatever later changes the layers" (F6's proposal).**
   - Problem: the operator's off switch no longer reaches a review already running.
   - Case: the operator sees a member PR-Review being settled by the orch and sets `autonomy=false` to step in. The session stays non-interactive, so there is no operator seat. The parent's satisfied answer still ends the review.
   - Evidence: the support log's accepted trade-off says "an operator who wants to turns autonomy off first", which now holds only before the session starts. D4's Bound names no way for the operator to take over mid-review, such as a send-back.
3. **LOW — D5 class 2, "a new task, since only the operator makes one", against the F9 rewording of Scope Out.**
   - Problem: class 2's reason is unchanged, but D3's raddle and D7's merge task now make tasks under autonomy. So "only the operator makes one" contradicts the rewritten Scope Out bullet and D7's Bound.
4. **LOW — D8, the orch posts each severe-halt relay and merge-ready notice to `<shortname>:SLACK` and also opens the turn reply with `[operator]` (F14's proposal).**
   - Problem: under autonomy with a slack presence, the hook forwards the marked reply and the direct post arrives too. Each severe halt and merge-ready notice would reach Slack twice.
   - Evidence: D8 requires both the marker and the direct post, and it gives no dedupe or rule that one of them is skipped.

Tool calls:
- grep of `internal/driftengine`, `internal/darnengine` and `orch-role.md` for added entries: my context lacked how drift reads the decision record's added entries.
- grep for `CheckAddedEntriesSubset` users and a read of `judgesnapshot.go:360-380`: my context lacked whether added entries feed the judge snapshot.
