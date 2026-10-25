```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals carried out as proposed, which are F1, F3 and F4. F2 was fixed differently, so it is out of this lens.

**MEDIUM: F4 keeps the record for errors raised before anything changed**
- Changed text: card 17, "Every other such error restarts no dobby … an error from the pass wait, the parent-lock acquisition, the dobby end or the log-sink close … each releases the parent lock when held, calls `Close` and returns the error naming the re-run".
- Summary: these errors leave the move record standing even when no content has moved, so they stop weft writers that never needed to stop.
- Case: on the prime, the operator interrupts the pass wait, or the parent-lock acquisition fails. Nothing has moved and no dobby has ended. The record still stands, so until someone runs the `--set` again:
  - card 13's refusals stop every weft pull, the push that must replay, Add, reconcile and checkout of the worktree;
  - card 16's gate stops every dobby of this machine.
- Evidence: the only reason the review's fix gives for keeping the record is that "card 16's gate refuses a start while the record stands". That reason disappears if the record is removed first. D7 step 4's failure rule (`_lyx/discussion/decision-record.md:147`) removes the record and restarts the dobbies step 2 ended, and nothing has been written by that point. An error before `Apply` is in the same state, so `RemoveMoveRecord`, a restart of any dobby already ended, then `Close` would put things back to how they were before the verb. Keeping the record is justified only for an `Apply` error that left it.

**MEDIUM: F1's way forward may not progress without a fetch**
- Changed text: card 17, "an unfinished status there names `lyx batten cancel <slug>` run on the owner machine … and then the `--set` … once the cancel's push has reached origin". Also card 15's `UpstreamRunUnfinished` "reads … at the tree of its weft's fetched upstream tip".
- Summary: the refusal reads the remote-tracking ref, and nothing named in the way forward refreshes that ref after the owner's cancel.
- Case: the owner cancels and pushes. On this machine the weft's tracking ref still points at the old tip, because the marker change stops the weft pull. The `--set` runs `UpstreamRunUnfinished` against that stale tip, reads the run as unfinished, and refuses with the same text again, which is the loop F1 set out to break.
- Evidence: card 17 names no fetch inside `changeAnchor` or `hubAnchorDeps`. Both the owner refusal and the new reader say "fetched upstream tip". The fix should either name a fetch step in the way forward, such as the weft pull that fetches before it stops (card 9), or have the verb fetch the weft before it reads.

F3: no findings. The tolerant remover with an allowlist entry follows the `failedpush.go` and `mergestate.go` precedent, and each of its three call sites removes the record only where the cards already prescribe removal.

Tool calls:
1. `grep` over `_lyx` for D7 and its failure-before-commit wording, to find the decision record and its text, which my context did not hold.
2. `sed -n 130,160p _lyx/discussion/decision-record.md`, to read D7's steps and failure rules, which my context did not hold.
